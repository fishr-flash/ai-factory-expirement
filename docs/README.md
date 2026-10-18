# AI Factory documentation

This page is the entry point for the AI Factory v1.0 product and engineering documentation. The corpus is intended for a team that has not participated in the design discussions.

## Sources of authority

1. [`specification.md`](specification.md) is the authoritative source for product requirements P-001–P-048 and acceptance scenarios A-01–A-15.
2. [`technical/`](technical/README.md) describes the current implementation contract for v1.0. It elaborates the requirements without reducing them.
3. [`technical/decisions.md`](technical/decisions.md) records approved architecture decisions and explicitly bounded implementation decisions. It does not authorize a scope change.
4. [`superpowers/`](superpowers/) contains the dated design, roadmap, verification and implementation-planning history. These files are preserved as design provenance. They are not a substitute for the technical contract; when an illustrative example or planning detail differs, consult the requirements and the current technical document, and record unresolved decision conflicts rather than inferring provider behavior.

The precedence rule is: immutable safety/platform constraints and human approvals; product acceptance requirements; role authority; allowlisted product-local instructions; general factory instructions; task-specific wording. The phrase “product-local instructions take precedence” never overrides the first three levels.

## Read by task

| Reader / task | Start here |
|---|---|
| New engineer | [Technical overview](technical/overview.md), then [system architecture](technical/architecture/system-architecture.md) |
| Implement native role workflows | [Agent operating model](technical/architecture/agent-operating-model.md), [Codex Cloud boundary](technical/platform/codex-cloud.md), [data contracts](technical/interfaces/data-contracts.md) |
| Implement persistence or recovery | [State and recovery](technical/architecture/state-and-recovery.md), [data contracts](technical/interfaces/data-contracts.md), [cross-repository saga](technical/interfaces/cross-repository-saga.md) |
| Implement Python tooling | [CLI contract](technical/interfaces/command-line-interface.md), [implementation guide](technical/implementation/implementation-guide.md) |
| Prepare or operate a Cloud run | [Environment setup](technical/operations/environment-setup.md), [production runbook](technical/operations/production-runbook.md), [incident and recovery](technical/operations/incident-recovery.md) |
| Verify v1.0 | [Requirements and acceptance](technical/verification/requirements-and-acceptance.md) |

## Document map

```text
docs/
├── README.md
├── specification.md
├── technical/
│   ├── README.md
│   ├── overview.md
│   ├── decisions.md
│   ├── architecture/
│   │   ├── system-architecture.md
│   │   ├── agent-operating-model.md
│   │   ├── state-and-recovery.md
│   │   ├── resource-management-and-self-correction.md
│   │   └── security-and-privacy.md
│   ├── interfaces/
│   │   ├── data-contracts.md
│   │   ├── command-line-interface.md
│   │   └── cross-repository-saga.md
│   ├── platform/
│   │   └── codex-cloud.md
│   ├── operations/
│   │   ├── environment-setup.md
│   │   ├── wp0-evidence.md
│   │   ├── production-runbook.md
│   │   └── incident-recovery.md
│   ├── verification/
│   │   └── requirements-and-acceptance.md
│   └── implementation/
│       └── implementation-guide.md
└── superpowers/  # preserved design and planning history
```

## Status and change control

This is the technical design baseline for AI Factory v1.0. It is documentation, not an implementation: no runtime behavior is claimed as delivered and no A-scenario is marked passed. Codex Cloud capabilities marked **documented** are statements from the linked official documentation; capabilities marked **runtime-unverified** require evidence in the user's Cloud account under WP0. A blocked WP0 capability requires a recorded human architecture decision; it does not permit an API runtime or manual child-task relay.

Changes to a product requirement require the product owner/customer's approval. Changes to accepted architecture, authority, privacy, billing model, runtime or v1.0 acceptance require an explicit architecture decision. Editorial and implementation detail may be refined if it preserves all P/A requirements and constraints. Update the relevant contract, decision/traceability links, and this index in the same change.

## Historical Superpowers artifacts

- [`superpowers/specs/2026-10-18-ai-factory-design.md`](superpowers/specs/2026-10-18-ai-factory-design.md)
- [`superpowers/specs/2026-10-18-ai-factory-roadmap.md`](superpowers/specs/2026-10-18-ai-factory-roadmap.md)
- [`superpowers/specs/2026-10-18-ai-factory-verification.md`](superpowers/specs/2026-10-18-ai-factory-verification.md)
- [`superpowers/plans/2026-10-18-ai-factory-v1-implementation.md`](superpowers/plans/2026-10-18-ai-factory-v1-implementation.md)

