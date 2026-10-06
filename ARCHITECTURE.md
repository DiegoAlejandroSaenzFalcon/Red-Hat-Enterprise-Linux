# RHEL Enterprise Lab — Architecture

## System boundary

The laboratory is a dual-boot RHEL environment on the Lenovo laptop. Windows remains the workstation environment. RHEL is intentionally minimal and headless.

Known hardware baseline from Windows inventory:

- Lenovo 82XB
- Intel Core i3-N305, 8 logical cores
- approximately 8 GB RAM
- approximately 512 GB SSD
- Intel integrated graphics

The exact RHEL baseline is captured only after booting RHEL.

## Layer model

L0 Hardware/Firmware → L1 RHEL Minimal → L2 Identity/Security → L3 Network/System Services → L4 Storage/Containers → L5 Development/Automation → L6 Observability/Incidents → L7 AI-assisted Operations → L8 Documentation/Evidence/Portfolio

## Capability domains

| Domain | Capability | Evidence |
|---|---|---|
| Base | RHEL installation, packages, users | baseline |
| Security | SELinux, firewalld, SSH, audit | security report |
| System | systemd, journald, timers | runbook evidence |
| Storage | LVM, filesystems, mounts, backup | storage lab |
| Network | NetworkManager, DNS, diagnostics | network lab |
| Containers | Podman, rootless execution | container evidence |
| Development | Git, Python, venv | reproducibility |
| Automation | Bash/Python | scripts + tests |
| Monitoring | CPU/RAM/disk/logs | monitoring evidence |
| Incidents | controlled failure/recovery | postmortems |
| AI | governed assistance | contracts + traces |
| Portfolio | Pages/docs | public documentation |

## Resource policy

No GUI is required. No always-on local LLM is required. No unnecessary databases or duplicate services. Measure before and after significant changes.

## Security zones

LOCAL ADMIN → controlled SSH/sudo/system administration

SERVICES → explicitly enabled + firewalld controlled + SELinux enforcing

CONTAINERS → rootless by default + least privilege + explicit network exposure

AI ASSISTANCE → read-only discovery by default + proposed changes + verification + human authorization for destructive operations

## Source-of-truth hierarchy

1. Platform and safety constraints.
2. Central ecosystem governance.
3. Repository contracts.
4. Specifications and architecture.
5. Runbooks and scripts.
6. Evidence from actual execution.
7. AI suggestions.

AI output never outranks observed system state.
