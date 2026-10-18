# Technical documentation

The technical corpus turns the approved AI Factory v1.0 requirements and architecture into component, persistence, interface, operation and verification contracts. It is organized by the questions implementers and operators need to answer, not by the order in which the design work was produced.

The technical prose is written in English to keep identifiers, module names and implementation-facing contracts directly aligned with code. The approved product requirement source remains in Russian at [`../specification.md`](../specification.md); its P- and A-IDs are preserved without translation or renumbering.

| Area | Documents | Boundary |
|---|---|---|
| Product/system orientation | [Overview](overview.md), [decisions](decisions.md) | Scope, vocabulary, constraints, architecture choices and unresolved evidence gates. |
| Architecture | [System architecture](architecture/system-architecture.md), [agent operating model](architecture/agent-operating-model.md), [state and recovery](architecture/state-and-recovery.md), [resources and self-correction](architecture/resource-management-and-self-correction.md), [security and privacy](architecture/security-and-privacy.md) | Runtime responsibilities and invariants. These documents do not claim any implementation exists. |
| Interfaces | [Data contracts](interfaces/data-contracts.md), [CLI](interfaces/command-line-interface.md), [two-repository saga](interfaces/cross-repository-saga.md) | Inputs, outputs, durable records, conflict semantics and component boundaries. |
| Platform | [Codex Cloud](platform/codex-cloud.md) | Officially documented behavior versus account-specific WP0 findings and hard blockers. |
| Operations | [Environment setup](operations/environment-setup.md), [WP0 evidence record](operations/wp0-evidence.md), [production runbook](operations/production-runbook.md), [incident recovery](operations/incident-recovery.md) | Human-operated environment and run procedures, including pause/resume. |
| Verification | [Requirements and acceptance](verification/requirements-and-acceptance.md) | P/A traceability, evidence expectations and release gates. |
| Implementation handoff | [Implementation guide](implementation/implementation-guide.md) | Package/module boundaries, build order, test layers and future implementation completion criteria. |

### Cross-document contract rules

- Requirement IDs always refer to [`../specification.md`](../specification.md); wording and acceptance scope are not redefined here.
- Durable state is Git-versioned in the factory and product repositories. The Codex task/session is an execution context, not the system of record.
- Native Codex subagents in one root Cloud task are the sole child-delegation mechanism. A separate Cloud task may be started by a human only as an authorized run recovery after VM loss; it is never a child-assignment relay.
- The Python CLI is a finite deterministic helper. It does not schedule, spawn agents, select work, call a model or create Cloud tasks.
- Platform-specific capability is not inferred from a local Codex client, a sample config or a different ChatGPT account. Such behavior remains **runtime-unverified** until WP0 evidence is recorded.

