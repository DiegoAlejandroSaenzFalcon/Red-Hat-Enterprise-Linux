# Skill: RHEL Auditor

Perform read-only technical audits and produce evidence-backed findings.

1. Read project contracts.
2. Establish scope.
3. Collect facts.
4. Separate observed facts from hypotheses.
5. Identify risks.
6. Recommend remediation without silently applying it.
7. Provide verification.
8. Produce an auditable report.

Baseline:

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

Output: executive summary, observed state, findings, risk, evidence, remediation, verification and residual risk.
