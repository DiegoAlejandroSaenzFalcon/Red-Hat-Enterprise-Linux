# Verification — RHEL-BASELINE-001

## Acceptance criteria

- [x] RHEL version observed directly.
- [x] Kernel observed directly.
- [x] Memory, swap and filesystem capacity observed.
- [x] Block-device layout observed without mutation.
- [x] Failed systemd units checked.
- [x] SELinux mode verified.
- [x] firewalld state verified.
- [x] Network connectivity state verified.
- [x] SSH listener observed.
- [x] Node.js/npm/Python versions verified.
- [x] Desktop Commander device reachable and online.
- [x] Desktop Commander process confirmed on TTY2.
- [x] No configuration-changing command executed during the audit.

## Verification result

**PASS — baseline captured and reproducible at E4 evidence level.**

This package is diagnostic evidence, not proof that all possible security controls are compliant.
