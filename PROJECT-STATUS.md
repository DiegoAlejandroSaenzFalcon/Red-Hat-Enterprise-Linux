# Project Status

**Project:** RHEL Enterprise Lab  
**Repository:** Red-Hat-Enterprise-Linux  
**Owner:** Diego Alejandro Saenz Falcon  
**Target:** RHEL 10.x Minimal / headless enterprise laboratory

## Repository state

- Existing RHEL 10.2 educational content: PRESENT
- Security guidance: PRESENT
- MkDocs configuration: PRESENT
- GitHub Pages target: PRESENT
- AI agent baseline: PRESENT
- Enterprise architecture: BEING RESTRUCTURED
- AI contracts: ADDED
- Evidence standard: ADDED
- Learning path: ADDED
- Laboratory matrix: ADDED

## Physical environment

RHEL installation media is currently being prepared on USB.

No claim is made about the final RHEL installation state until RHEL is booted and inspected.

## Next verification gate

After first RHEL boot:

    cat /etc/redhat-release
    uname -a
    hostnamectl
    lsblk -f
    findmnt
    df -hT
    free -h
    getenforce
    systemctl --failed
    systemctl --type=service --state=running
    firewall-cmd --state
    nmcli general status
    dnf repolist

The output becomes Baseline 0.

## State model

USB_PREPARATION → BOOT_VERIFICATION → INSTALLATION → BASELINE_0 → SECURITY_BASELINE → LAB_BUILD → VERIFICATION → PORTFOLIO

Do not infer RHEL state from Windows. Do not repartition or overwrite the SSD without observed installer information and an explicit storage decision.


## SOLUCIONATIA-001 — Wi-Fi RHEL 10.2

**Fecha:** 2026-10-06  
**Estado:** VERIFIED / EVIDENCED / DOCUMENTED

- Incidencia: interfaz Intel `wlp0s20f3` sin gestión por NetworkManager.
- Causa confirmada: ausencia de `wpa_supplicant` y `NetworkManager-wifi`.
- Recuperación efectiva: paquetes oficiales mediante DNF usando USB tethering temporal.
- Contingencia offline: RHEL USB + `images/install.img` inspeccionado y documentado.
- Verificación: NetworkManager reiniciado y Wi-Fi operativo.
- Evidencia: `evidence/2026-10-06/SOLUCIONATIA-001/`.
- Alcance cerrado: únicamente recuperación Wi-Fi; trabajos posteriores quedan fuera de este caso.


## RHEL-BASELINE-001 — Auditoría técnica inicial

**Fecha:** 2026-10-06  
**Estado:** EVIDENCED / DOCUMENTED

Primera línea base técnica del RHEL físico actual, obtenida directamente mediante Desktop Commander Remote en modo de observación.

- RHEL: 10.2 (Coughlan)
- Kernel: 6.12.0-211.62.1.el10_2.x86_64
- Target: multi-user.target
- Unidades systemd fallidas: 0
- SELinux: Enforcing
- firewalld: activo, zona public
- SSH: activo, TCP/22 escuchando
- Wi-Fi: operativo
- Node.js: 22.23.2 / npm 10.9.8
- Python: 3.12.14
- Desktop Commander: online en TTY2, versión 0.2.52
- RAM disponible observada: ~6.1 GiB de 7.2 GiB
- /: 5% usado; /home: 3% usado

### Hallazgos pendientes

- RTC configurado en hora local: evaluar antes de cambiarlo por coexistencia con Windows.
- Medio de instalación montado: no desmontar ni modificar sin autorización.
- DNF reporta repositorios de Subscription Management no actualizados: auditar registro/repositorios.
- SSH expuesto en TCP/22: revisar política y exposición antes de habilitar acceso remoto móvil.
- Desktop Commander sigue en modo interactivo TTY2; la persistencia como servicio queda pendiente de diseño y autorización.

**Evidencia:** [evidence/2026-10-06/RHEL-BASELINE-001/](./evidence/2026-10-06/RHEL-BASELINE-001/)


## AICCP-ARCHITECTURE-v1 — Diseño del nodo operativo

**Fecha:** 2026-10-06  
**Estado:** SPECIFIED / DOCUMENTED

La Fase 1 confirmó que RHEL será el nodo operativo gobernado para ChatGPT, Desktop Commander, OpenCode, auditoría y Git/GitHub.

- RAM observada: ~7.2 GiB total / ~6.0 GiB disponible.
- Swap: 7.6 GiB, sin uso.
- Git: **2.52.0 instalado y verificado en /usr/bin/git.**
- GitHub CLI: no detectado.
- OpenCode/AICCP: no activos.
- Desktop Commander: activo exclusivamente en TTY2; TTY1 permanece libre.
- Node.js 22.23.2 y Python 3.12.14 disponibles.
- Workspace creado en `/home/DevFS/workspaces/`.
- Repositorio `Red-Hat-Enterprise-Linux` clonado localmente en el workspace.
- No se instalarán componentes pesados hasta superar los gates de diseño y medición.

Diseño: `docs/AICCP-ARCHITECTURE-v1.md`.

### Gate C — Bootstrap mínimo

**Estado:** IMPLEMENTED / VERIFIED / EVIDENCED / DOCUMENTED

- Git 2.52.0 instalado por el operador con privilegios administrativos.
- Verificación remota: `git version 2.52.0`.
- Binario verificado: `/usr/bin/git`.
- No se configuraron credenciales Git globales.
- `gh` aún no está instalado.
- Workspace RHEL creado y repositorio canónico clonado.
- No se instaló OpenCode, AICCP ni infraestructura pesada durante este gate.

**Evidencia:** [evidence/2026-10-06/RHEL-GATE-C-001/](./evidence/2026-10-06/RHEL-GATE-C-001/)

### Próximo gate

Validar acceso Git/GitHub para los repositorios requeridos y mapear el workspace antes de instalar OpenCode. La configuración de proveedor/modelo de OpenCode se descubrirá después de la instalación y sin introducir API keys.
