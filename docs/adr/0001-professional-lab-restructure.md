# ADR-0001 — Professional RHEL Lab Restructure

## Status

Accepted

## Context

The repository began as a didactic RHEL knowledge base. The wider ecosystem now requires a stronger infrastructure project that can function as:

- a real practice environment;
- a reproducible laboratory;
- a professional portfolio;
- an AI-governed infrastructure node.

## Decision

Evolve the repository into a layered Enterprise Linux laboratory rather than replacing the existing guides.

The existing guides and security reports remain historical/technical assets and are integrated progressively into the new architecture.

The lab will use:

- minimal/headless RHEL;
- native RHEL security controls;
- systemd and operational tooling;
- Podman;
- Python/Bash automation;
- evidence-driven exercises;
- explicit AI contracts and skills;
- GitHub Pages for presentation.

## Consequences

Positive:

- stronger professional signal;
- reproducible learning;
- safer AI-assisted operations;
- clearer relationship between implementation and evidence;
- reusable runbooks and incident practice.

Trade-off:

- the repository becomes larger and more structured;
- historical material must be reconciled rather than blindly treated as current truth.

## Rejected alternative

Keep the repository as a collection of isolated tutorials.

Reason for rejection: it teaches individual commands but does not demonstrate system lifecycle, governance, operations or professional ownership.
