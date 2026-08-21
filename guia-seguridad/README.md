# Guía: Auditoría y Endurecimiento de Seguridad en RHEL 10.2

### Diego Alejandro Saenz Falcon · Red Hat Enterprise Linux 10 (RHEL 10.2) · Ciberseguridad

> **Reporte de seguridad profesional** estructurado sobre la obra
> *Enciclopedia de la Seguridad Informática* (Álvaro Gómez Vieites, 2ª ed., RA-MA, 2011)
> y respaldado por una auditoría técnica real con **OpenSCAP** usando los baselines
> oficiales **CIS Red Hat Enterprise Linux 10** y **DISA STIG RHEL 10**.

---

## 0. Metodología y principios (Bloque 1 — principios de la SI)

La seguridad se aborda desde la **tríada CIA** (Confidencialidad, Integridad,
Disponibilidad) y un **SGSI** (Sistema de Gestión de Seguridad de la Información)
por capas:

1. **Reconocimiento** (qué tenemos y qué exponemos).
2. **Análisis de vulnerabilidades** (manual + OpenSCAP CIS/STIG).
3. **Endurecimiento** (cierre de huecos con lo disponible en el sistema).
4. **Auditoría y respuesta** (trazabilidad, evidencia, reversibilidad).

> **Marco de madurez** (de la enciclopedia): sentido común -> adecuación legal ->
> gestión integral -> certificación ISO 27000. Este entorno parte de "sentido común +
> cumplimiento CIS/STIG verificable" sin instalar servicios de larga duración.

---

## 1. Resultados de la auditoría técnica (OpenSCAP)

Herramientas: `scap-security-guide-0.1.81` + `openscap-scanner-1.1.4.4` (instaladas
solo para la prueba; **no dejan demonios persistentes**: `oscap` corre bajo demanda).

| Baseline | Evaluados | Pass | Fail | Nota |
|----------|-----------|------|------|------|
| **CIS RHEL 10 — Server L1 (línea base, antes)** | 293 | 173 | 120 | 59% cumplimiento |
| **CIS RHEL 10 — Server L1 (después de endurecer)** | 293 | 185 | 108 | 63% (+12) |
| **DISA STIG RHEL 10 (línea base)** | 465 | 176 | 280 | 9 sin comprobar |
| **DISA STIG RHEL 10 (después de endurecer)** | 465 | 190 | 266 | 9 sin comprobar |

Los reportes HTML completos (por regla, con CCE y remediación sugerida) están en
esta carpeta: `rhel10-cis-l1-report.html`, `rhel10-cis-l1-after-report.html`,
`rhel10-stig-report.html`. Para regenerarlos:

```bash
DS=/usr/share/xml/scap/ssg/content/ssg-rhel10-ds.xml
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_server_l1 \
  --report rhel10-cis-l1-report.html --results rhel10-cis-l1-results.xml "$DS"
```

### Hallazgos críticos y de alto riesgo (antes del endurecimiento)

| Severidad | Hallazgo | Estado tras endurecimiento |
|-----------|----------|----------------------------|
| Critico | `sshd` con `PermitRootLogin yes` + `PasswordAuthentication yes` | `sshd` **desactivado** (se reactiva solo con llave, ver §6) |
| Alto | `DevFS ALL=(ALL) NOPASSWD: ALL` (sudo sin contraseña ni traza) | Reemplazado por sudo con `log_input`/`log_output` + contraseña |
| Alto | Sin bloqueo por fuerza bruta (`pam_faillock` ausente) | `pam_faillock` activo (`deny=3`, `even_deny_root`) |
| Medio | Política de contraseñas inexistente (caducidad 99999 días) | `PASS_MAX_DAYS=90`, `PASS_MIN_DAYS=1`, `chage` aplicado |
| Medio | firewalld abierto: `cockpit` + `ssh` en ambas interfaces | `cockpit` y `ssh` **eliminados** de la zona public |
| Medio | `bluetooth.service` activo sin uso | Desactivado |
| Bajo/Residual | `wpa_supplicant` copiado manualmente (no rpm) | Documentado (ver §5); pendiente de `dnf install` en entorno gestionado |
| Bueno | SELinux `Enforcing`, un solo UID 0, `openssh`/plugins parcheados, `/etc/shadow` 0000, sin `ld.so.preload`, `mitigations` ON | Mantenido |

---

## 2. Identificación y autenticación (Bloque 3)

### 2.1 sudo "pro" (mínimo privilegio + auditoría)
Se eliminó el `NOPASSWD: ALL` ciego. El nuevo `/etc/sudoers.d/devfs`:

```sudoers
Defaults log_input, log_output
Defaults iolog_dir=/var/log/sudo-io
DevFS ALL=(ALL) ALL
```

Cada comando sudo queda registrado (entrada/salida) en `/var/log/sudo-io` ->
**trazabilidad total** sin instalar nada.

### 2.2 Bloqueo por fuerza bruta (PAM)
`authselect enable-feature with-faillock` + `/etc/security/faillock.conf`:

```ini
audit
silent
deny = 3
fail_interval = 900
unlock_time = 600
even_deny_root
root_unlock_time = 600
```

### 2.3 Política de contraseñas
`/etc/login.defs` ajustado y aplicado con `chage` a `root` y `DevFS`:
caducidad 90 días, mínimo 1 día entre cambios, aviso 7 días.

---

## 3. Criptografía y manejo de secretos (Bloque 4)

- **Cero secretos en el repositorio**: `SECURITY.md` y `HONEYTOKEN.md` fijan la
  regla. Ninguna clave privada, token ni password se escribe en este repo.
- El token de GitHub vive en `~/.config/gh/hosts.yml` con permisos `0600` (fuera
  del repo).
- Para acceso remoto se usará **autenticación por llave pública** (§6), nunca
  contraseña ni clave privada en chat.

---

## 4. Medidas técnicas en red (Bloque 5 — firewall / inalámbrico)

- **firewalld**: zona `public` reducida a `dhcpv6-client` únicamente; se eliminaron
  `cockpit` y `ssh`.
- **Endurecimiento sysctl** (`/etc/sysctl.d/99-security.conf`):
  `kernel.dmesg_restrict=1`, `kernel.kptr_restrict=1`,
  `net.ipv4.conf.*.rp_filter=1`, `accept_redirects=0`, `send_redirects=0`,
  `accept_source_route=0`, `fs.protected_hardlinks=1`, `fs.protected_symlinks=1`.
- **Inalámbrico**: la WiFi funciona vía `wpa_supplicant` recuperado offline (ver
  `../guia-wifi-rhel10/README.md`). **Advertencia de integridad** en §5.

---

## 5. Nota de integridad: el binario `wpa_supplicant` manual

El `wpa_supplicant` y el plugin WiFi de NetworkManager se copiaron a mano desde el
ISO de instalación porque en este entorno **no se instaló nada vía `dnf`** (restricción
del propietario). Consecuencia:

- Estos binarios **no están gestionados por RPM** (no aparecen en `rpm -V`, no se
  actualizan con `dnf update`). Es un **hueco de integridad** conocido.
- **Cierre recomendado en entorno gestionado** (con acceso a repositorios):
  ```bash
  sudo dnf install -y wpa_supplicant NetworkManager-wifi
  sudo reboot
  ```
  Esto reemplaza los binarios manuales por paquetes firmados y verificables.

---

## 6. Reactivación de SSH (cuando se necesite) — método seguro

SSH está **apagado**. Para volver a usarlo desde el celular u otro equipo, usar
**llave pública** (nunca contraseña, nunca clave privada en chat):

```bash
# 1) Generar par en el servidor (sin passphrase si es para automatización del celular)
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_rhel
# 2) Publicar la pública
cat ~/.ssh/id_ed25519_rhel.pub >> ~/.ssh/authorized_keys
# 3) Mostrar la CLAVE PRIVADA solo en esta terminal local (NO en el chat):
cat ~/.ssh/id_ed25519_rhel
# 4) Copiarla a mano en la app SSH del celular (Termius/JuiceSSH/ConnectBot).
# 5) Rehabilitar sshd y abrirlo SOLO para la IP del celular en firewalld:
sudo systemctl enable --now sshd
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --add-rich-rule='rule family=ipv4 source address=IP_DEL_CELULAR service name=ssh accept' --permanent
sudo firewall-cmd --reload
```

`sshd_config` recomendado (endurecido, built-in):
`PermitRootLogin no`, `PasswordAuthentication no`, `PubkeyAuthentication yes`,
`MaxAuthTries 3`, `LoginGraceTime 20`, `MaxStartups 10:30:60`, `AllowUsers DevFS`.

---

## 7. Opción A (con instalación) vs Opción B (sin instalación)

El propietario pidió **no instalar nada salvo el tooling de auditoría**. Esto define
dos posturas documentadas:

### Opción A — Entorno gestionado / con instalación (lo que usan los profesionales)
| Herramienta | Qué es | ¿Deja servicio 24/7? |
|-------------|--------|----------------------|
| `scap-security-guide` + `openscap-scanner` | Baseline CIS/STIG/PCI para auditar | No (corre bajo demanda) |
| `aide` | Integridad de archivos (FIM) | No (programado) |
| `fail2ban` | Bloqueo de IPs por fuerza bruta en host expuesto | Sí (demonio) — típico en host único a Internet |
| `tlog` / session-recording | Grabación de sesiones (entornos regulados) | Sí (servicio deliberado) |

Los profesionales **instalan lo que la política de seguridad y el cumplimiento
exigen, documentado**; no instalan "por si acaso".

### Opción B — Entorno restringido / sin instalación (lo hecho aquí)
Todo el endurecimiento se logró con lo ya presente: `sshd`, `sudo` (I/O logging),
PAM (`pam_faillock`), `auditd`, `firewalld`, `nmcli`, `sysctl`. **Cero paquetes
nuevos de runtime.** El único paquete instalado fue el de auditoría (Opción A
acotada), que no añade servicios.

---

## 8. Cumplimiento y marco legal (Bloque 7)

- Evidencia de cumplimiento verificable: reportes **CIS** y **DISA STIG** incluidos.
- Protección de datos: política de **cero secretos** en el repo; trazabilidad vía
  sudo I/O-log + `auditd`.
- Reversibilidad: cada cambio es deshacible (perfiles `authselect`, `firewall-cmd
  --remove`, `/etc/sysctl.d` aislados).

---

## 9. Comandos de verificación rápida

```bash
sudo systemctl is-enabled sshd bluetooth.service        # esperado: disabled
sudo authselect current                                  # with-faillock activo
faillock --user DevFS                                    # estado de bloqueo
sudo firewall-cmd --list-all                             # solo dhcpv6-client
sudo sysctl kernel.dmesg_restrict kernel.kptr_restrict   # 1 / 1
sudo oscap xccdf eval --profile ... --report rep.html "$DS"
```

---

*Autor: Diego Alejandro Saenz Falcon* · https://github.com/DiegoAlejandroSaenzFalcon
