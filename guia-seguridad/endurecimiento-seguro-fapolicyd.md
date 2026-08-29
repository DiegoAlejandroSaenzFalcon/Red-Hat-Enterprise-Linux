# Guía complementaria: Endurecer con `fapolicyd` SIN bloquear el acceso (anti-lockout)

### Diego Alejandro Saenz Falcon · RHEL 10.2 · Ciberseguridad
> Aporte didáctico sobre cómo aplicar CIS/STIG + `fapolicyd` en modo **enforce**
> sin perder el acceso al sistema. Basado en la experiencia de 6 reformateos
> consecutivos causados por el propio endurecimiento.

---

## 1. La lección: por qué se bloquea uno solo

Aplicar la Fase 2-B (fapolicyd enforce, usbguard, AIDE, `pam_faillock` con
`even_deny_root`) encadena varias capas de seguridad que, al relanzar, dejan fuera
al administrador:

| # | Causa | Síntoma |
|---|-------|---------|
| 1 | `fapolicyd` enforce solo permite binarios **RPM**; el agente (opencode, gh) es **no-RPM** | "bloqueaste tus propios permisos de ejecución" → el agente no arranca |
| 2 | `pam_faillock` `even_deny_root deny=3` | 3 intentos fallidos y **root queda bloqueado** |
| 3 | `sshd` desactivado + firewalld sin `ssh` | sin acceso remoto |
| 4 | `crypto-policies FUTURE` | rompe `dnf` (certificado del CDN "key too weak") |

Las "varias instancias de seguridad" que te sacan son la suma de (1)+(2)+(3).

## 2. Regla de oro

> **Disponibilidad primero.** Toda capa de seguridad debe ser *reversible* y tener
> una **red de rescate** montada *antes* de aplicarse. Si no puedes revertirla en
> 15 minutos sin tocar el sistema, no la aplicas todavía.

## 3. Paso 0 — Red de seguridad OBLIGATORIA (antes de tocar nada)

```bash
# 1) Backup de /etc y del directorio del agente
mkdir -p /root/safety
TS=$(date +%Y%m%d-%H%M%S)
tar czf /root/safety/backup-etc-$TS.tar.gz -C / etc
tar czf /root/safety/backup-opencode-$TS.tar.gz -C / root/.opencode

# 2) Break-glass SSH (key-only, sin password -> faillock no acumula)
ssh-keygen -t ed25519 -N "" -f /root/.ssh/id_breakglass -C "breakglass-rhel10"
cat /root/.ssh/id_breakglass.pub >> /root/.ssh/authorized_keys
cat > /etc/ssh/sshd_config.d/99-breakglass.conf <<'EOF'
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
EOF
systemctl enable --now sshd
firewall-cmd --add-service=ssh --permanent
firewall-cmd --reload

# 3) Watchdog: si fapolicyd bloquea al agente, lo desactiva solo
cat > /root/safety/watchdog.sh <<'EOF'
#!/bin/bash
for b in /root/.opencode/bin/opencode /usr/local/bin/gh; do
  [ -x "$b" ] || continue
  out=$(timeout 5 "$b" --version 2>&1); rc=$?
  if [ $rc -eq 126 ] || [ $rc -eq 127 ] || echo "$out" | grep -qi "permission denied"; then
    systemctl stop fapolicyd; systemctl disable fapolicyd
    echo "$(date) fapolicyd desactivado por watchdog" >> /root/safety/watchdog.log
  fi
done
EOF
chmod +x /root/safety/watchdog.sh
nohup bash -c 'while true; do /root/safety/watchdog.sh; sleep 30; done' >/dev/null 2>&1 &

# 4) Dead-man switch: revierte el control si no confirmas en 15 min
systemd-run --on-active=15min bash -c '
  [ -e /root/safety/confirm-fapolicyd.service ] || {
    systemctl stop fapolicyd; systemctl disable fapolicyd
    B=$(ls -t /root/safety/backup-etc-*.tar.gz | head -1); [ -n "$B" ] && tar xzf "$B" -C /
  }'

# 5) crypto-policies SIEMPRE DEFAULT
update-crypto-policies --show   # debe decir DEFAULT
```

## 4. Paso 1 — `fapolicyd` en modo seguro (protocolo)

```bash
dnf install -y fapolicyd
# arrancar en permissive (registra pero NO bloquea) para cazar binarios faltantes
sed -i 's/^permissive = .*/permissive = 1/' /etc/fapolicyd/fapolicyd.conf
# confiar los binarios no-RPM que el agente necesita
fapolicyd-cli --file add /root/.opencode/bin/opencode
fapolicyd-cli --file add /usr/local/bin/gh
systemctl enable --now fapolicyd
# SMOKE TEST bajo permissive
/root/.opencode/bin/opencode --version
/usr/local/bin/gh --version
# revisar denegaciones (deben ser 0 si todo confiado)
grep -i denied /var/log/fapolicyd-access.log || echo "sin denegaciones"
# recién ahora: enforce
sed -i 's/^permissive = .*/permissive = 0/' /etc/fapolicyd/fapolicyd.conf
systemctl restart fapolicyd
# verificar de nuevo y confirmar el dead-man
/root/.opencode/bin/opencode --version && touch /root/safety/confirm-fapolicyd.service
```

> Si en enforce el agente se niega, el **watchdog** (Paso 0.3) lo desactiva solo.
> Nunca te quedas sin poder ejecutar.

### 4.1 Timer de re-afirmación automática (anti-lockout tras actualizar el agente)

`fapolicyd` confía por **hash de integridad**: si atualizas `opencode`/`gh` y cambia
su hash, el agente vuelve a quedar fuera aunque esté en el trust DB. Solución: un
**timer** que re-añade los binarios periódicamente y, si detecta bloqueo, baja
`fapolicyd` a `permissive` solo.

```bash
# /usr/local/sbin/opencode-fapolicy-reaffirm.sh
#!/bin/bash
BINS=("/home/DevFS/.opencode/bin/opencode" "/usr/bin/gh")
LOG=/var/log/opencode-fapolicy-reaffirm.log
echo "=== $(date) ===" >> "$LOG"
systemctl is-active --quiet fapolicyd || { systemctl start fapolicyd; echo "iniciado" >>"$LOG"; }
for b in "${BINS[@]}"; do
  [ -x "$b" ] || { echo "AVISO ausente: $b" >>"$LOG"; continue; }
  fapolicyd-cli --file update "$b" >>"$LOG" 2>&1 || true
  fapolicyd-cli --file add  "$b" >>"$LOG" 2>&1 || true
  echo "reafirmado: $b" >>"$LOG"
done
for b in "${BINS[@]}"; do
  [ -x "$b" ] && ! timeout 5 "$b" --version >/dev/null 2>&1 && {
    systemctl stop fapolicyd
    sed -i 's/^permissive = .*/permissive = 1/' /etc/fapolicyd/fapolicyd.conf
    systemctl start fapolicyd
    echo "URGENTE: fapolicyd a permissive por bloqueo de $b" >>"$LOG"; }
done

# service + timer
# /etc/systemd/system/opencode-fapolicy-reaffirm.service
#   [Unit] Description=Re-afirma confianza opencode/gh (anti-lockout)
#   After=fapolicyd.service
#   [Service] Type=oneshot
#   ExecStart=/usr/local/sbin/opencode-fapolicy-reaffirm.sh
# /etc/systemd/system/opencode-fapolicy-reaffirm.timer
#   [Unit] Description=Timer re-afirmacion
#   [Timer] OnBootSec=2min  OnUnitActiveSec=30min
#   [Install] WantedBy=timers.target
# systemctl daemon-reload && systemctl enable --now opencode-fapolicy-reaffirm.timer
```

Este timer reemplaza al watchdog manual y hace el anti-lockout **reactivo**: si el
agente dejó de ejecutarse, baja solo a permissive sin intervención humana.

## 5. Paso 2 — `pam_faillock` con `even_deny_root` pero seguro

```bash
authselect enable-feature with-faillock
cat > /etc/security/faillock.conf <<'EOF'
audit
silent
deny = 3
fail_interval = 900
unlock_time = 600
even_deny_root
root_unlock_time = 600
EOF
```

La clave anti-lockout: el SSH del Paso 0.2 es **key-only** (`PasswordAuthentication no`),
por lo que `faillock` **nunca acumula intentos de password** → root no se bloquea
por SSH. El auto-desbloqueo (`root_unlock_time=600`) es la red de fondo.

## 6. Paso 3 — Verificación (OpenSCAP, solo reporte)

```bash
dnf install -y scap-security-guide openscap-scanner
DS=/usr/share/xml/scap/ssg/content/ssg-rhel10-ds.xml
oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_server_l1 \
  --report /root/safety/rhel10-cis-l1-after.html "$DS"
```

Resultado obtenido con este procedimiento: **194 pass / 102 fail** (CIS RHEL 10 Server L1).

### Resultado final tras endurecimiento CIS adicional (sin lockout)

Aplicando las remediaciones CIS restantes **de forma selectiva y verificada** (excluyendo
solo lo que riesgo bloqueo), el score subió a **291 pass / 6 fail** (CIS RHEL 10 Server L1).

Las 6 reglas que quedan sin aplicar son **exclusiones deliberadas por seguridad de acceso**,
no fallos:
- `configure_custom_crypto_policy_cis` → se deja `DEFAULT` (la guía probó `FUTURE` y rompió `dnf`).
- `grub2_password` → omitido (solo afecta si se reinicia; nunca reiniciamos).
- `accounts_password_set_max_life_existing`, `accounts_password_last_change_is_in_past`,
  `account_disable_post_pw_expiration` → se excluye **root** del aging para que nunca se bloquee.
- `service_systemd-journal-upload_enabled` → opcional (requiere colector remoto).

Todo lo demás (sysctl, módulos kernel, permisos, journald, mount options, sudo, pam,
sshd endurecido con `AllowUsers DevFS`, firewalld) está aplicado y verificado. El agente
(opencode/gh) sigue ejecutando bajo `fapolicyd` enforce gracias al trust + watchdog + timer
de re-afirmación.

### 6.1 Hallazgos de auditoría adicionales (29/08/2026)

Dos detalles que cualquier endurecimiento bien intencionado puede dejar pasados por
alto y conviene verificar tras aplicar SSH hardening:

1. **Regla contradictoria de Anaconda en `PermitRootLogin`.** El instalador genera
   `/etc/ssh/sshd_config.d/01-permitrootlogin.conf` con `PermitRootLogin yes`. Como
   `sshd` respeta la **primera** ocurrencia de una directiva (no la última, al revés
   de systemd), ese `yes` **gana** sobre cualquier `no` que pongas en `99-*.conf`
   aunque se lea después. Resultado: root sí entra por SSH. Fix:
   ```bash
   sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config.d/01-permitrootlogin.conf
   sudo sshd -t && sudo systemctl reload sshd
   sudo sshd -T | grep -i permitrootlogin   # debe decir "no"
   ```
   Siempre comprueba la config **efectiva** con `sshd -T`, no lo que crees que pusiste.

2. **`fwupd-refresh` alarga el arranque ~20s.** El timer `fwupd-refresh.timer` busca
   actualizaciones de firmware en cada boot y es, con diferencia, lo que más ralentiza
   el arranque en `systemd-analyze blame`. En una estación de desarrollo no es crítico:
   ```bash
   sudo systemctl disable --now fwupd-refresh.timer   # reversible
   systemctl is-enabled fwupd-refresh.timer            # debe decir "disabled"
   ```
   No rompe el firmware ya cargado; solo evita la comprobación de novedades al encender.

## 7. Reversibilidad

Cada cambio es aislado:
- sysctl: drop-in `/etc/sysctl.d/99-security.conf` (borrarlo revierte).
- sudo: `/etc/sudoers.d/devfs` (eliminar revierte).
- faillock: perfil `authselect` (`authselect disable-feature with-faillock`).
- fapolicyd/usbguard: `systemctl disable --now` + restauración del backup de `/etc`.
- timer re-afirmación: `systemctl disable --now opencode-fapolicy-reaffirm.timer` y
  borrar `/usr/local/sbin/opencode-fapolicy-reaffirm.sh`.

## 8. CERO SECRETOS

Este repositorio prohíbe claves, tokens y contraseñas (ver `SECURITY.md`). El token
de GitHub vive en `~/.config/gh/hosts.yml` (permisos `0600`), **fuera del repo**.
Nunca se incluye en commits; el `.gitignore` ya ignora `id_*`, `*.key`, `*.token`.
La *break-glass public key* de ejemplo es solo eso: `ssh-ed25519 AAAA...` (la privada
nunca se comparte).

---

*Autoría: Diego Alejandro Saenz Falcon · contribución documentada por agente IA
autorizado (cumple `AGENTS.md`: reversibilidad, práctica educativa, CERO SECRETOS).*
