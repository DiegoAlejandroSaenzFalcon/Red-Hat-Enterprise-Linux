# Repository Restructure Map

## Before

The repository was centered on three independent guides:

- WiFi recovery.
- Performance optimization.
- Security auditing/hardening.

This material remains useful.

## After

The repository becomes a lifecycle-oriented laboratory:

BASE → SECURITY → SYSTEM → STORAGE → NETWORK → DEVELOPMENT → CONTAINERS → AUTOMATION → MONITORING → INCIDENTS → AI OPERATIONS → EVIDENCE → PORTFOLIO

## Content migration rule

Existing material is not deleted merely because the architecture changed.

Each legacy guide is classified as:

- KEEP — technically current and directly reusable.
- RECONCILE — useful but must be checked against the new baseline.
- HISTORICAL — valuable as evidence of previous experiments.
- RETIRE — technically obsolete or unsafe.

## Existing security evidence

The OpenSCAP CIS/STIG reports are valuable historical evidence and must be preserved. Their results must not be presented as the current state of the newly installed RHEL host until the new host is audited.

## Migration principle

Preserve knowledge, improve structure, verify reality.
