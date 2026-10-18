# Durable data contracts

## Contract purpose and encoding

This document defines the minimum records exchanged by production roles and the Python helper. Product requirements remain pinned to [`../../specification.md`](../../specification.md). Durable records use JSON; append-only event and resource streams use JSONL, as decided in [D-10](../decisions.md). This selects one concrete encoding for v1.0 because the older design contains illustrative YAML paths while the later implementation plan selected JSON/JSONL. It does not alter product behavior.

Implementation must provide versioned JSON Schema Draft 2020-12 definitions for all record types below. Each record has `schema_version`; unknown keys are rejected except inside named extension objects. Schema validation is followed by cross-record validation because JSON Schema cannot prove repository refs, parent closure, uniqueness across a journal or acyclic task dependencies. Schemas are packaged with the Python distribution and loaded without network access.

## Repository layout and ownership

```text
factory/roles/<role>.md                     # versioned role base prompts
factory/policies/<policy>.md                # authority, delegation, privacy, approvals
.agents/skills/<skill>/SKILL.md             # reusable Codex procedures
factory/models.json                          # verified capability catalog, unknowns allowed
config/factory.json                          # validated policy defaults; no credentials
factory/templates/{manifest,assignment,decision,experiment}.json
production/active/<project_id>/<run_id>/
  manifest.json
  assignments/<task_id>/assignment.v<N>.json
  executions/<agent_id>.jsonl
  decisions/<decision_id>.json
  resources/ledger.jsonl
  sync/<operation_id>.json
  acceptance/<acceptance_id>.json
production/archive/<project_id>/<run_id>/     # moved only after final state and evidence
experiments/<experiment_id>.json              # authorized experiment evidence
config/factory.json                            # validated operational defaults + WP0 capability snapshot
factory/models.json                            # versioned evidence-based model capability catalog
```

Project/run/task/agent-specific records are scoped under the run. Separate record files avoid unrelated agents writing the same file. The manifest is a compact current snapshot; journals are append-only evidence. Corrections append superseding records and preserve the original Git history. Product code, private source excerpts and product acceptance criteria stay in the product repository; the factory stores pinned refs/hashes and the minimum authorized evidence.

## Common rules

- **IDs:** stable opaque identifiers with a type prefix (for example `run_…`, `task_…`, `agent_…`, `event_…`, `op_…`). IDs are unique within their declared namespace, never derived from timestamp alone, immutable after publication, and never reused after cancellation/deletion.
- **Time:** ISO 8601 UTC timestamp with `Z`; ordering comes from committed event sequence/refs, not timestamps alone.
- **Hashes:** full Git object SHA for a commit/ref or cryptographic content hash for a file. Record algorithm or use the repository's Git object format explicitly; do not accept abbreviated display hashes as write preconditions.
- **Refs:** each Git repo reference includes repository ID (sanitized remote or configured logical ID), branch/ref and full commit SHA. A product repo and factory repo are separate refs.
- **Unknown:** numeric observation is `null` and paired with a status/reason/source such as `status: "unknown"`, `source: "unavailable"`. Zero means a measured zero only.
- **Privacy:** records carry `privacy_class`; public journal fields cannot contain copied confidential source or credentials. Redaction findings and publication approval do not make the original content public-safe automatically.
- **Actor:** `actor_id`, role, and authority reference; a model name alone is not authority.
- **References:** every `*_ref` resolves to a record, immutable source artifact, full Git SHA, or explicitly `unknown` value; validation reports unresolved references.

## Run manifest

**Path:** `production/active/<project_id>/<run_id>/manifest.json`. **Owner:** PO. **Update:** expected-SHA compare-before-write; all meaningful changes also have a journal/decision reference.

| Field | Type / rule |
|---|---|
| `schema_version` | string; supported schema identifier |
| `project_id`, `run_id` | immutable typed IDs |
| `privacy_class` | configured enum; at minimum `public` or `private` |
| `source_spec` | `{repo_ref, path, sha256, approved_by, approved_at}`; immutable for this run; criteria changes require external approval and a new source version |
| `factory_repo`, `product_repo` | repo refs `{repo_id, remote_ref, branch, start_sha, current_sha}`; remote is sanitized and may be withheld if sensitive |
| `org` | active org tree, rationale, effective role/agent IDs, selected model/effort ref or unknown, WP0 capability ref |
| `cloud_task_ref` | Provider task identifier/link when observable; otherwise explicit unknown. It is not a recovery source. |
| `pinned_versions` | factory commit, role/policy/skill hashes, model catalog hash, product-local instruction hash and source commit |
| `state` | run FSM enum; each change references transition event and evidence |
| `requirement_coverage` | map of P/A requirement IDs to status, work/acceptance refs and gap reason |
| `resource_policy` | parent budget ref, units, concurrency cap source and unknown allowance behavior |
| `active_assignments` | immutable assignment IDs/versions and status refs |
| `open_sync_operations` | operation refs which are unresolved |
| `next_action` | explicit recovery-safe action, responsible role and prerequisite/evidence refs |
| `updated_at`, `last_event_ref` | timestamp and last persisted journal event/commit ref |

## Assignment

**Path:** `assignments/<task_id>/assignment.v<N>.json`. **Owner:** assigning parent. **Update:** immutable once supplied to a child; changes create a new version. One assignment has one parent/authority chain.

Required content: `schema_version`, `assignment_id`, `assignment_version`, `run_id`, `task_id`, `parent_task_id`, `assignee_role`, `agent_id`, `agent_profile_ref`, `requirement_ids`, `objective`, `scope`, `allowed_context_refs`, `allowed_paths`, `dependencies`, `deliverables`, `acceptance_checks`, `authority_snapshot`, `pinned_instruction_refs`, `skill_refs`, `requested_model`, `requested_reasoning_effort`, `observed_model`, `observed_reasoning_effort`, `agent_thread_ref`, `parent_thread_ref`, `resource_reservation`, `retry_policy`, `stop_conditions`, `escalation_conditions`, `created_by`, `created_at`, `supersedes_ref`.

Invariants:

- `requirement_ids` are a non-empty subset of parent requirements or explicit authorized enabling work. A parent-child closure check finds any omitted requirement.
- `dependencies` refer to existing task IDs and form an acyclic graph.
- Each deliverable has path/repository scope and evidence expectations. Acceptance checks are observable and cannot weaken the pinned product criterion.
- `allowed_context_refs` and `allowed_paths` are enforceable by parent review/branch policy only to the degree the platform permits; they are not a sandbox claim.
- Model fields state requested and observed values separately. Use `{status: "unknown", value: null, reason}` when not observable.
- Agent and parent thread refs are recorded when Cloud exposes them; absent or unobservable IDs are explicit unknowns, not generated substitutes.
- `resource_reservation` includes parent allocation ID, resource type/unit/window, amount, status and source. It is a soft factory limit.
- Retry policy includes maximum attempts, attempt count, same-failure review threshold and exhausted-budget outcome.
- Stop/escalation criteria name reason, receiving authority and required evidence.

## Events, decisions and execution journals

All are append-only JSONL unless the record represents an immutable snapshot.

| Record | Required fields |
|---|---|
| `ExecutionEvent` | `schema_version`, `event_id`, `run_id`, `project_id`, `task_id`, `agent_id`, `role`, `assignment_ref`, `event_type`, `occurred_at`, `actor`, `source_refs`, `payload`, `supersedes_ref` (nullable). Worker payload is factual: action, check, result, failure/retry, artifact commit/PR ref, status. No self-improvement analysis in a Worker event. |
| `Decision` | `schema_version`, `decision_id`, `run_id`, `decision_type`, `scope`, `actor`, `authority_ref`, `rationale`, `alternatives`, `evidence_refs`, `outcome`, `approval_requirement`, `timestamp`, `supersedes_ref` (nullable). |
| `ResourceEvent` | `schema_version`, `event_id`, `run_id`, `parent_task_id`, `child_task_id`, `reservation_id`, `resource_type`, `unit`, `window`, `estimate`, `reserved`, `observed`, `status`, `source`, `reliability`, `remaining_allowance`, `stop_decision`, `timestamp`, `supersedes_ref` (nullable). |

Every resource quantity is `{value, unit, status, source, observed_at}`; `status` is one of `estimated`, `reserved`, `observed`, `unknown`. Unknown has null value and a reason. Resource events record reservations, consumption observations, release, correction, pause/stop and the authority behind any exception. A correction links to the prior event and leaves it intact.

## Sync operation

**Path:** `sync/<operation_id>.json`. **Owner:** PO or authorized parent. Updated by immutable intent/result versions or append-only events.

Required fields: `schema_version`, `operation_id`, `idempotency_key`, `run_id`, `assignment_ref`, `operation_type`, `factory_expected_sha`, `product_expected_sha`, `intent`, `status`, `product_result_ref`, `factory_result_ref`, `attempts`, `reconciliation`, `created_at`, `updated_at`. Status is `INTENT_RECORDED`, `PRODUCT_APPLIED`, `FACTORY_RECORDED`, `RECONCILED`, `CONFLICT`, or `BLOCKED`. The factory intent must be durable before product mutation; unknown outcome must not be replayed. See [saga protocol](cross-repository-saga.md).

## Acceptance record

**Path:** `acceptance/<acceptance_id>.json`. Required fields: `schema_version`, `acceptance_id`, `run_id`, `assignment_ref` or `product_scope`, `acceptance_level` (`LOCAL` or `PRODUCT`), immutable `criteria_refs`, `evidence_refs`, `reviewer_id`, `reviewer_role`, `decision` (`ACCEPTED`, `CHANGES_REQUESTED`, `REJECTED`), `gaps`, `timestamp` and any superseded acceptance ref. Local reviewer is the direct parent. Product reviewer is PO. Product acceptance points to external specification/version and integration evidence; tests alone are insufficient.

## Experiment record

**Path:** `experiments/<experiment_id>.json`. Required fields: `schema_version`, `experiment_id`, `scope` (`product` or `factory`), `state`, `hypothesis`, `baseline_ref/sha`, `candidate_ref/sha`, `author`, `authorizer`, `authority_ref`, `comparison_cohort`, `stratification`, `success_metrics`, `guardrails`, `sample_size`, `limitations`, `rollout_scope`, `rollback_trigger`, `evaluation_refs`, `decision`, `decision_at`. States: `OBSERVATION`, `PROPOSAL`, `AUTHORIZED_EXPERIMENT`, `EVALUATION`, then one of `ADOPTED`, `REJECTED`, `ROLLED_BACK`. Adopted versions apply to future work; assignments in progress retain their pins. Rollback creates another immutable revision.

## Cross-record validation

The validator must reject: duplicate IDs; incorrect ID namespace; missing/mismatched run/project/repository refs; unresolved assignment parent/requirement/evidence refs; dependency cycles; child requirement loss; acceptance by wrong role; state transition without evidence; an assignment whose pinned policy differs from run pin without explicit restart decision; unknown usage encoded as zero; result before saga intent; duplicate idempotency key with a different operation; a manifest inconsistent with journal/head refs; public record containing configured secret canary or forbidden privacy fields.

Successful schema validation is necessary but not sufficient for acceptance or recovery. Consumers must validate cross-record invariants and current Git SHAs before acting.

## Factory configuration and model catalog

These files are operational contracts as well as configuration, so they require their own JSON Schemas and cross-validation. They are not records of inferred provider capability.

### `config/factory.json`

Required fields: `schema_version`, `privacy_classes`, `retry_defaults`, `fanout_defaults`, `org_defaults`, `cloud_capabilities`, `usage_policy`, `protected_actions_ref`, and `updated_from_evidence_ref`.

- `privacy_classes` defines allowed values and storage/redaction rules. Initial v1.0 values include public and private.
- `retry_defaults` and `fanout_defaults` supply ceilings that assignments may tighten but not silently loosen. Per-task constraints and WP0-tested session limits also apply.
- `org_defaults` describes preferred shapes/required minimum roles; it cannot force an optional manager into a run.
- `cloud_capabilities` stores evidence refs and values for max nesting, max open child threads, supported delegation method, model/reasoning controls, permission inheritance and task recovery. Every value has a status (`runtime_verified`, `documented`, `unknown`, `unsupported`) and evidence ref. A missing/unknown required value blocks the dependent behavior.
- `usage_policy` defines observed/estimated/unknown handling, stop threshold source and resume evidence requirement. It must not claim a provider-enforced per-run hard cap.
- No credentials, network secret values, personal identifiers or private product source may appear.

### `factory/models.json`

Required fields: `schema_version`, `provider_surface` (the approved `codex_cloud` surface), `verified_at`, `official_sources`, `wp0_evidence_ref`, and `models` array. Each model entry contains `model_id`, `availability_status`, `reasoning_efforts` (known list or explicit unknown), `capabilities`, `limitations`, `evidence_ref`, `verified_at` and optional `role_guidance`.

Model availability is per signed-in identity/product surface, may change, and must be checked against Codex Cloud rather than copied from an API catalog or local Codex config. Use no guessed model ID or capability. A catalog entry marked documented but not runtime-verified cannot by itself authorize use in a Cloud role. Missing model visibility is `unknown`, not a synonym for an unavailable model.

Both files reject unknown fields and are validated at environment setup, run initialization and before model-dependent delegation. Updating them requires a new factory revision and provenance; assignments retain requested and observed model/effort separately.

