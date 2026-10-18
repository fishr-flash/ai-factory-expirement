# Implementation handoff guide

## Purpose

This document gives an implementation team component boundaries, source layout, dependency order and verification responsibilities. It is not an executable implementation plan and contains no product code. Use the [CLI contract](../interfaces/command-line-interface.md), [data contracts](../interfaces/data-contracts.md), [agent model](../architecture/agent-operating-model.md), and [acceptance matrix](../verification/requirements-and-acceptance.md) as the behavioral contract. The original detailed Superpowers plan remains available as historical planning provenance at [`../../superpowers/plans/2026-10-18-ai-factory-v1-implementation.md`](../../superpowers/plans/2026-10-18-ai-factory-v1-implementation.md).

## Source layout target

```text
src/ai_factory/
  __init__.py       # version
  __main__.py       # python -m entry
  cli.py            # argparse/output/exit mapping only
  commands.py       # finite synchronous command use cases
  contracts.py      # typed values/errors shared by modules
  validation.py     # JSON/JSONL + cross-record validation
  ids.py            # stable typed IDs
  state.py           # pure run/work-item FSM
  run_store.py      # factory-side run files/checkpoint/recovery load
  gitops.py         # safe Git refs/SHA/saga reconciliation
  ledger.py         # append-only resource reservations/events
  pause.py          # pause checkpoint/resume preconditions
  organization.py   # explainable tree recommendation, no spawning
  assignments.py    # contract split and requirement closure
  acceptance.py     # acceptance evidence/authority validation
  redaction.py      # deterministic candidate redaction/findings
  privacy.py        # privacy/protected action policy checks
  metrics.py        # reports from validated events
  experiments.py    # authorized prompt/skill lifecycle validation
  schemas/          # packaged Draft 2020-12 JSON Schemas
    manifest.schema.json
    assignment.schema.json
    event.schema.json
    decision.schema.json
    resource-event.schema.json
    sync-operation.schema.json
    acceptance.schema.json
    experiment.schema.json
    factory-config.schema.json
    model-catalog.schema.json
```

`pyproject.toml` exposes `ai-factory` and `python -m ai_factory`, targets Python 3.12 and packages all ten schemas. The last two schemas validate `config/factory.json` and `factory/models.json` as specified in [data contracts](../interfaces/data-contracts.md#factory-configuration-and-model-catalog). Runtime logic remains in ordinary reviewable modules; no module owns background lifetime or a task queue.

## Module responsibilities and dependency direction

| Module | Depends on | Must not do |
|---|---|---|
| `contracts` | Standard library types | Persist data or embed policy decisions. |
| `ids` | Standard library | Use timestamps alone for uniqueness. |
| `validation` | Contracts, packaged schemas, pure record checks | Write records, call Git/Cloud/model. |
| `state` | Contracts | Decide which transition an agent wants; it only validates evidence/transition. |
| `organization` | Contracts, Cloud capability snapshot | Spawn subagents or choose unavailable models. |
| `assignments` | Contracts, validation | Call Codex; it verifies requirements, DAG, criteria, resource and authority fields. |
| `run_store` | Validation, Git ref service | Mutate product repo or overwrite expected SHA conflict. |
| `gitops` | Local Git executable | GitHub API, PR creation, product content mutation or force push. |
| `ledger` / `pause` | Contracts, state, run store | Claim plan hard cap or auto-resume after reset. |
| `acceptance` | Contracts, validation | Accept work; it validates whether the supplied reviewer/evidence can support a recorded decision. |
| `privacy` / `redaction` | Contracts/config | Treat token-pattern redaction as proof of safe publication. |
| `experiments` / `metrics` | Validated event stream | Adopt prompt changes autonomously or interpret a single incident as cross-product evidence. |
| `commands` | The application modules above | Make an orchestration loop; call exactly the synchronous use case requested. |
| `cli` | `commands`, output renderer | Choose work/model/tree, launch Cloud tasks, call provider APIs or store hidden state. |

Dependency direction is inward toward contracts and pure domain checks. CLI is the outermost adapter. Native Codex roles remain the production coordinator; Python functions are invoked by explicit role/operator instructions and return before the agent decides what to do next.

## Public typed values and errors

Use `pathlib.Path` for filesystem paths and immutable typed values for results. Minimum shared types:

- `Diagnostic(path, line, code, message)`;
- `ValidationReport(valid, diagnostics, checked_records)`;
- `RepoRef(repo_id, branch, head_sha, sanitized_remote)`;
- `RunRef(project_id, run_id, run_dir)` and `RunSnapshot(manifest, open_assignments, open_operations, refs)`;
- `WriteResult(changed_paths, expected_sha, resulting_sha)`;
- `OrgPlan(tree, rationale, requirement_refs)`;
- `Reservation(parent_id, child_id, unit, amount, status)` and `ReservationDecision(allowed, reason, observed_limit, evidence_ref)`;
- `AuthorizationDecision(allowed, actor, required_authority, evidence_ref, reason)`;
- `RedactionReport(candidate_text, findings, publication_review_required)`;
- `ReconcileReport(operations, observed_refs, proposed_actions, blocked)`;
- `Report(requirement_coverage, acceptance, resources, decisions, next_action)`;
- typed errors `ContractValidationError`, `ConflictError`, `PolicyError`, `BlockedError`.

The data contract document is the persisted JSON authority; Python return values do not become a second disk format.

## Build and work-package order

Follow the existing approved roadmap; the order below groups implementation responsibility, not a scope reduction:

| WP | Implementation focus | Effort | Main risk | Main modules/docs | Exit evidence |
|---|---|---:|---|---|---|
| WP0 | Account/environment/subagent feasibility | 2–3 engineer-weeks | Cloud hierarchy, concurrency, model and two-repo behavior may not meet requirements. | [Cloud contract](../platform/codex-cloud.md), [WP0 evidence](../operations/wp0-evidence.md) | Required topology, repo access and model/usage behavior pass or an explicit human blocker is recorded. No Cloud-dependent code before pass. |
| WP1 | Versioned schemas, IDs, FSM and policy foundations | 2–4 | Parallel writers, schema evolution and usage evidence are account-level. | `schemas`, `validation`, `ids`, `state`; [data](../interfaces/data-contracts.md), [state](../architecture/state-and-recovery.md) | Valid/invalid contract cases, stable diagnostics, FSM tests and Git-only recovery model. |
| WP2 | Git run store and two-repo reconciliation | 3–5 | Partial writes, competing refs and ambiguous side effects. | `run_store`, `gitops`; [saga](../interfaces/cross-repository-saga.md) | Expected-SHA conflicts, partial/duplicate/ambiguous side-effect integration tests. |
| WP3 | Native roles, config/skills, org and delegation | 4–6 | Cloud may not load local project role config or permit required nesting/model choices. | `AGENTS.md`, `.codex/*`, `.agents/skills/*`, `organization`, `assignments` | Cloud-tested native role tree and assignment closure. No cross-task relay. |
| WP4 | Ledger, retry, pause/resume | 3–5 | No per-run hard cap; plan usage and task consumption vary. | `ledger`, `pause`; [resources](../architecture/resource-management-and-self-correction.md) | Shared soft reservations, no-fan-out-on-unknown policy, durable pause/recovery. |
| WP5 | Local/product acceptance, escalation and reporting | 2–4 | Local test success can be mistaken for product completion. | `acceptance`, `metrics`, command integration | Reject incomplete criteria; separate local vs product acceptance. |
| WP6 | Governed instruction experiments | 3–5 | Confounding, low sample size, active prompt drift. | `experiments`, experiment schema/template | Authority chain, baseline/cohort evidence, pinned active assignment, adoption/rejection/rollback. |
| WP7 | Privacy, secret handling and protected actions | 3–5 | Prompt injection, public metadata leakage, broad credentials. | `privacy`, `redaction`, policies | Threat-model tests, redaction findings, protected-action denial and publication review. |
| WP8 | Full multi-product acceptance and independent review | 4–6 | Rare failures and incomplete evidence. | Acceptance suites, operator/release docs | All A-01–A-15/P-001–P-048 evidence, security sign-off and release approval. |

Dependencies are WP1 after WP0; WP2 after WP1; WP3 after WP0–WP2; WP4 after WP1–WP3; WP5 after WP1–WP4; WP6 after WP1/WP3/WP5; WP7 after WP0–WP6; WP8 after WP0–WP7. Parallel work is safe only after the relevant contracts are fixed; shared manifest writes must be serialized by SHA check.

Direct effort totals 26–43 engineer-weeks. Add 30–40% contingency for Cloud capability findings, integration, native-subagent coordination and repeat acceptance; planning range is 34–61 engineer-weeks. With two to three core engineers plus part-time security/reliability review and product-owner availability for external approvals, the calendar estimate is 14–23 weeks. Confidence is low (approximately ±40%) until WP0 passes; external-access and plan-reset waiting time is excluded. The estimate uses the approved roadmap but is not a commitment to a reduced or partial release.

## Testing and quality expectations for implementation

This guide does not create or run tests. The implementation team should use `unittest` with isolated temporary directories and temporary Git repositories:

- **Unit:** schema diagnostics, ID uniqueness/validation, FSM, requirement closure, org selection, reservations/retries, privacy and action authority, experiment transitions, metrics classification.
- **Integration:** CLI help/exit codes/output, init/checkpoint, JSONL append, recovery, expected-SHA conflict, each saga interruption and idempotent reconciliation.
- **Role contract:** parse TOML, required agent fields, role-to-skill links, skill metadata, no unauthorized self-acceptance/prompt change and no separate-task relay.
- **Acceptance:** one named repeatable scenario per A ID; Cloud-only behavior is marked pending until actual account evidence exists.
- **Security/privacy:** secret canaries, malicious product instructions, write scope behavior, private/public record review, approval gates and no force-push/API path.

Initial local validation commands after implementation are `python -m pip install -e .`, `python -m unittest discover -s tests -v`, and `python -m unittest discover -s tests/acceptance -v`. They are future implementation commands and have not been run as part of this documentation work.

Every externally visible command must have deterministic behavior, explicit preconditions, stable exit mapping, privacy-safe diagnostics and an expected-SHA rule for mutations. No “test passed” or release state is inferred from documentation alone.

## Implementation completion definition

The product is implemented only when all WP0–WP8 exit conditions pass; required Cloud topology is evidenced inside one root task; both repositories support the verified workflow; Git-only recovery works; accepted product output meets external criteria; the controlled factory experiment lifecycle works without Worker self-governance; all P/A coverage has retained evidence; no unresolved critical/high release blocker remains; independent reviewer and authorized owner approve the release. Finishing a module or passing local tests is not v1.0 acceptance.

