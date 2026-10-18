# Architecture decision register

This register restates current decisions so implementers can find them without mining discussion history. The underlying approved design and chronology remain in [`../superpowers/specs/2026-10-18-ai-factory-design.md`](../superpowers/specs/2026-10-18-ai-factory-design.md). An item marked **WP0 gate** is intentionally unresolved and must not be treated as supported.

| ID | Decision | Consequence / evidence |
|---|---|---|
| D-01 | Factory and each product are separate GitHub repositories; Git files are durable state; no production database or transactional cross-repo store. | Use per-repository refs and a saga with recovery evidence. See [system architecture](architecture/system-architecture.md), [data contracts](interfaces/data-contracts.md), [saga](interfaces/cross-repository-saga.md). |
| D-02 | v1.0 runtime is Codex Cloud through the ChatGPT subscription, not OpenAI API billing. | No API key, Agents/Responses API client, external model adapter or API spend. Account access and included-plan behavior must be verified for the intended identity. |
| D-03 | Human starts and resumes the root task through the approved ChatGPT UI workflow. | Codex Cloud commands/automation, even if documented by the platform, are excluded from the product runtime until an explicit scope decision. No autonomous reactivation after quota reset. |
| D-04 | Native Codex subagents, under one root task, are the only child-delegation mechanism. | Manual relay of child assignments between separate Cloud tasks is prohibited. A human may start a recovery task under the same run ID after the old VM is unavailable; it must reconstruct state from Git and is not ordinary delegation. |
| D-05 | PO/Chief/Planner/Worker is a dynamic role tree; PO and at least one Worker are required, leadership is optional and justified. | Preserve flat, PO→Planner, PO→Chief→multiple Planner and, where natively supported, PO→multiple Planner structures. Required multi-direction A-03 topology remains a release gate. |
| D-06 | Role authority is hierarchical; human owns PO core, immutable safeguards and protected approvals. | Task wording may change within delegated scope; role-base prompt changes require direct parent approval, provenance and experiment. |
| D-07 | Provider sessions are temporary; Git state and checkpoints are authoritative. | Resume same task when available; after VM loss, operator starts recovery task with same run identity; quota reset does not auto-resume. |
| D-08 | Resource reservations are conservative soft controls, not a subscription-level hard cap. | No new fan-out when current headroom cannot be evidenced under the approved policy; record usage as unknown if the UI does not expose it. |
| D-09 | No per-agent OS sandbox/path isolation is assumed. | Subagents inherit root permissions; context/path scope is contractual and behavior-reviewed; Cloud/repository permission boundary is the actual enforcement surface. |
| D-10 | Durable record serialization for v1.0 is JSON for individual records and JSONL for append-only event/usage journals. | This resolves a documentation inconsistency: the earlier design included illustrative YAML paths, while the later implementation plan specified JSON/JSONL and JSON Schema. No business requirement changes. New contracts use JSON/JSONL. |
| D-11 | The Python CLI is an optional deterministic helper; Codex agents own production orchestration. | CLI accepts one finite command and exits. It cannot spawn/dispatch agents, choose work/models/acceptance, start Cloud tasks or maintain a daemon/queue/service. |
| D-12 | WP0 is a hard Cloud capability gate and does not change acceptance scope. | Record verified depth, concurrency, model behavior, repository operations, access and usage visibility. Unsupported required behavior blocks v1.0 for human decision. |

## Open decisions and evidence gates

The following are not implementation defaults. Record actual evidence in [`platform/codex-cloud.md`](platform/codex-cloud.md) and the WP0 artifact; use `unknown` when it cannot be observed.

| Question | Required evidence | If unsupported |
|---|---|---|
| Does the selected account have Codex Cloud under the ChatGPT plan? | Plan/feature access in the actual account | Stop before runtime build that depends on it; return blocker/options. |
| Can one environment include both factory and product repositories with separate durable writes? | Cloud task performs disposable branch/commit or PR operations in both; identities, SHAs and recovery are recorded | Stop cross-repo production scope; no hidden copy or API workaround. |
| Can required native hierarchy and simultaneous thread count execute in one root task? | Exercise PO→Chief→2 Planners→Workers and inspect concurrent child threads | Stop at WP0; no separate Cloud task relay or reduced topology. |
| Can the root and child agents select/inherit the needed model/reasoning? | Actual available model control and observable model for each level | If architecture requires unsupported distinctions, stop for human decision; never fabricate a model record. |
| Which repository instructions/config/skills are loaded in Cloud? | Cloud task records loaded paths and behavior for AGENTS, project TOML/custom agents, skills | Use only verified generic instructions in the same root task, if all required role behavior remains possible; otherwise blocker. |
| What usage/headroom can an operator observe and attribute? | Usage UI and session/thread evidence; usage boundary/reset behavior | Treat per-run values as unknown, keep soft reservations, pause on policy threshold. |
| What recovery/interruption behavior applies? | Same-task reopen and saved VM retention in the actual account | Git-only recovery under same run ID remains mandatory. |

## Evidence labels

- **Documented:** present in a current official source for the named product surface.
- **Runtime-verified:** observed in the intended Cloud account and recorded with timestamp, environment, task/thread reference and non-sensitive evidence.
- **Runtime-unverified:** documented for another surface or not yet tested in Cloud.
- **Unsupported:** current official documentation explicitly says the capability is unavailable for the relevant surface.
- **Unknown:** evidence could not establish the fact. Unknown is not a pass.

