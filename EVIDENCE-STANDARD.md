# Evidence Standard

## Purpose

Distinguish professional evidence from documentation or claims.

## Evidence package

evidence/YYYY-MM-DD/LAB-ID/

- README.md
- commands.txt
- output.txt
- verification.md
- metadata.yaml

## Minimum metadata

lab_id, date, host, rhel_version, kernel, operator, objective, change_class, status, verification, rollback

Never store secrets or sensitive host identifiers.

## Evidence levels

- E0 Claim
- E1 Procedure
- E2 Execution
- E3 Verification
- E4 Reproducible evidence
- E5 Professional demonstration

Major capabilities should target E4/E5.

Evidence must support the exact claim being made.
