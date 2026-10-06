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
