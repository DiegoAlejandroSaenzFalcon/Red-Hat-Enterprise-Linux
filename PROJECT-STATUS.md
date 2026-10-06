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
