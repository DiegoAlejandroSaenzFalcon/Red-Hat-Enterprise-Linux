# Laboratory Matrix

| ID | Domain | Exercise | Risk | Verification | Target |
|---|---|---|---|---|---|
| BASE-001 | Base | Minimal RHEL installation | High | baseline | E4 |
| BASE-002 | Base | Host identity | Low | OS/hostname checks | E3 |
| SEC-001 | Security | SELinux enforcing | Medium | getenforce | E4 |
| SEC-002 | Security | firewalld baseline | Medium | firewall-cmd | E4 |
| SEC-003 | Security | SSH hardening | High | sshd validation | E4 |
| SYS-001 | System | systemd service lifecycle | Medium | systemctl | E4 |
| SYS-002 | System | journald diagnostics | Low | journalctl | E3 |
| STO-001 | Storage | LVM exercise | High | lvs/vgs/pvs | E4 |
| NET-001 | Network | NetworkManager diagnostics | Medium | nmcli | E4 |
| DEV-001 | Development | Git + Python environment | Low | reproducibility | E4 |
| CTR-001 | Containers | Rootless Podman | Medium | podman | E4 |
| AUT-001 | Automation | Idempotent admin script | Medium | repeated execution | E5 |
| MON-001 | Monitoring | Resource baseline | Low | metrics | E4 |
| INC-001 | Incident | Controlled service failure | High | recovery | E5 |
| INC-002 | Incident | Controlled network failure | High | recovery | E5 |
| AI-001 | AI | Read-only system audit | Low | evidence report | E5 |
| AI-002 | AI | Governed change proposal | Medium | approval + verify | E5 |
| DOC-001 | Portfolio | Publish architecture | Low | Pages build | E4 |

No row is complete without evidence.
