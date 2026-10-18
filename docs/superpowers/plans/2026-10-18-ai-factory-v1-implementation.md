# AI Factory v1.0 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Follow the native Codex subagent tree defined by the approved architecture. Do not hand off role assignments through separate Codex Cloud tasks.

**Goal:** Deliver the complete AI Factory v1.0: a reusable GitHub factory that runs product work in Codex Cloud under a ChatGPT subscription, delegates through native Codex subagents, durably records decisions and execution in Git, and improves its production methods through authorized evidence-based experiments.

**Architecture:** The factory is a versioned repository of role instructions, policies, schemas, Python tooling and run records; product code and product acceptance criteria stay in separate product repositories. A human starts or resumes one root Codex Cloud task in the ChatGPT UI; the PO role orchestrates a native Codex subagent tree in that task, while Git files are the durable state and the Python CLI validates and reconciles records. No API key, Agents/Responses API, custom server, database, or manual child-task relay is part of v1.0.

**Tech Stack:** Python 3.12; `jsonschema>=4.23,<5` for Draft 2020-12 validation (the [official docs](https://python-jsonschema.readthedocs.io/en/stable/) document Draft 2020-12 support); JSON for manifests/contracts/decisions; append-only JSONL for journals and ledgers; Markdown for Codex role instructions, policies and runbooks; `unittest` for tests; Git/GitHub for durable state, branches and PRs; Codex Cloud and native subagents through the ChatGPT subscription UI.

---

## Authority, gates, and execution rules

- Product authority: [`docs/specification.md`](../../specification.md), requirements P-001–P-048.
- Approved architecture: [`2026-10-18-ai-factory-design.md`](../specs/2026-10-18-ai-factory-design.md). Acceptance trace: [`2026-10-18-ai-factory-verification.md`](../specs/2026-10-18-ai-factory-verification.md). This plan does not amend either document.
- Native Codex configuration facts are based on [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents) and [Build skills](https://learn.chatgpt.com/docs/build-skills): project custom agents are TOML files, each requires `name`, `description`, and `developer_instructions`; per-role model/reasoning can be inherited or overridden; child agents inherit root permissions/provider/MCP; repository skills use a folder with `SKILL.md`. Cloud loading/effect still requires WP0 runtime evidence.
- WP0 is a hard prerequisite for implementation choices that depend on Cloud behavior. It must record the available nested depth, maximum simultaneously open child threads, model/reasoning inheritance and overrides, Cloud-visible agent configuration, parent permission/tool inheritance, actual account entitlement, two-repository writes, environment secrets and plan-usage evidence. Six child threads is the initial target to verify, not an assumed provider guarantee.
- If WP0 cannot run PO→Chief→Planner→Worker nesting or the required concurrency/model setup in one root Cloud task, stop at the gate and return evidence and options for a human architecture decision. Do not implement a separate-Cloud-task handoff, API path, or quiet reduction of P/A acceptance.
- During execution, use TDD for application behavior: for each named test, write one focused failing test, run that exact test, implement the smallest change, rerun it, then run the full suite at the task gate. Do not batch all failing tests before implementation. Keep commits small and task-scoped. Commands below are planned future commands; no implementation or test is performed while this plan is reviewed.
- Every Cloud subagent gets a written assignment contract and role instruction reference. Subagents inherit root permissions; path restrictions are enforced through assignment, review, branch policy and behavior checks, not represented as a per-agent OS sandbox.

## Repository and component map

This plan establishes the following implementation layout. The product repository is selected per run and is never nested inside the factory repository as a permanent product-code directory.

| Path | Responsibility |
|---|---|
| `pyproject.toml` | Installable Python package, Python version, runtime dependency and CLI entry point |
| `src/ai_factory/__init__.py` | Package version |
| `src/ai_factory/__main__.py` | `python -m ai_factory` entry point |
| `src/ai_factory/cli.py` | Thin argument-parsing adapter; no domain decisions, agent spawning, scheduling or provider API calls |
| `src/ai_factory/commands.py` | Synchronous use cases called by the CLI and Codex role instructions; validates, records and reconciles local Git state only |
| `src/ai_factory/contracts.py` | Shared typed value objects (`Diagnostic`, `ValidationReport`, `RepoRef`, `RunRef`, `RunSnapshot`, `WriteResult`, `ReconcileReport`, `OrgPlan`, `Reservation`, `ReservationDecision`, `AuthorizationDecision`, `RedactionReport`, `Report`, `CommandResult`) and domain exceptions |
| `src/ai_factory/validation.py` | Schema loading, JSON/JSONL parsing, schema and cross-record validation diagnostics |
| `src/ai_factory/ids.py` | Stable ID generation and validation; no generated ID is reused |
| `src/ai_factory/state.py` | Run/work-item state transition validation and transition events |
| `src/ai_factory/run_store.py` | Run creation, immutable record paths, Git checkpoint load/write and recovery reconstruction |
| `src/ai_factory/ledger.py` | Append-only event writes, budget reservations, resource reconciliation and concurrency reservation checks |
| `src/ai_factory/gitops.py` | Git ref inspection, compare-before-write, operation fingerprints, two-repository saga and recovery reconciliation |
| `src/ai_factory/organization.py` | Explainable PO/Chief/Planner/Worker tree selection under requirements and WP0 Cloud capabilities |
| `src/ai_factory/assignments.py` | Requirement-preserving child assignment split, dependencies, path/context boundaries and resource checks |
| `src/ai_factory/acceptance.py` | Direct-parent task acceptance and PO integration acceptance records |
| `src/ai_factory/pause.py` | Safe quota pause/checkpoint and authorized resume preconditions |
| `src/ai_factory/redaction.py` | Deterministic public-journal redaction and leak findings; never claims redaction proves no disclosure |
| `src/ai_factory/privacy.py` | Privacy-class handling and protected-action authorization checks |
| `src/ai_factory/metrics.py` | Requirement coverage and stratified quality/resource summaries from validated journals |
| `src/ai_factory/experiments.py` | Authorized prompt/skill experiment lifecycle and comparison results |
| `src/ai_factory/schemas/manifest.schema.json` | Draft 2020-12 run manifest contract |
| `src/ai_factory/schemas/assignment.schema.json` | Requirement-linked child assignment contract |
| `src/ai_factory/schemas/event.schema.json` | Append-only journal event contract |
| `src/ai_factory/schemas/decision.schema.json` | Decision, authority and evidence contract |
| `src/ai_factory/schemas/resource-event.schema.json` | Resource estimate/reservation/observation event contract |
| `src/ai_factory/schemas/sync-operation.schema.json` | Two-repository saga intent/result contract |
| `src/ai_factory/schemas/acceptance.schema.json` | Local and product-level acceptance contract |
| `src/ai_factory/schemas/experiment.schema.json` | Prompt/skill experiment lifecycle contract |
| `config/factory.json` | Defaults, privacy classes, retry/fan-out ceilings and model catalog schema reference; no secrets or API credentials |
| `.codex/config.toml` | Cloud-tested native subagent enablement and per-session open-thread cap; no provider override or credentials |
| `.codex/agents/chief.toml` | Codex custom-agent metadata/model settings for Chief, pointing to canonical role instructions |
| `.codex/agents/planner.toml` | Codex custom-agent metadata/model settings for Planner, pointing to canonical role instructions |
| `.codex/agents/worker.toml` | Codex custom-agent metadata/model settings for Worker, pointing to canonical role instructions |
| `AGENTS.md` | Root-thread PO instructions, native delegation policy and links to canonical factory policies |
| `factory/roles/po.md` | Root-thread PO authority, workflow, input/output contract and escalation protocol |
| `factory/roles/chief.md` | Chief authority, workflow, input/output contract and escalation protocol |
| `factory/roles/planner.md` | Planner authority, workflow, input/output contract and escalation protocol |
| `factory/roles/worker.md` | Worker authority, workflow, input/output contract and escalation protocol |
| `factory/policies/authority.md` | Role authority and parent-child prompt governance |
| `factory/policies/delegation.md` | Native subagent creation, nesting, result protocol and no cross-task handoff |
| `factory/policies/acceptance.md` | Local versus product acceptance authority and required evidence |
| `factory/policies/protected-actions.md` | Human approvals for consequential operations |
| `factory/policies/privacy.md` | Privacy classes, redaction and public/private repository rules |
| `.agents/skills/start-run/SKILL.md` | Codex skill for run intake, immutable requirement pinning and organization decision |
| `.agents/skills/delegate/SKILL.md` | Codex skill for assignment creation and native subagent spawn protocol |
| `.agents/skills/review-result/SKILL.md` | Codex skill for direct-parent result review and bounded retry process |
| `.agents/skills/integrate/SKILL.md` | Codex skill for PO integration, requirement coverage and final acceptance |
| `.agents/skills/recover-run/SKILL.md` | Codex skill for Git-only reconciliation, pause/resume and recovery |
| `factory/models.json` | Versioned model capability catalog with source/verification date and `unknown` handling |
| `factory/templates/manifest.json` | Valid initial run record |
| `factory/templates/assignment.json` | Valid assignment record |
| `factory/templates/decision.json` | Valid organizational/protected-action decision record |
| `factory/templates/experiment.json` | Valid experiment proposal record |
| `production/active/` and `production/archive/` | Git-durable run directories, separated by project/run and privacy class |
| `experiments/` | Authorized prompt/skill/organization experiments and outcome evidence |
| `tests/unit/` | Deterministic Python unit tests using temporary directories and fake Git repositories; package marker allows dotted unittest commands |
| `tests/integration/` | CLI, JSONL, run recovery and two-repository saga integration tests using local temporary Git repos; package marker allows dotted unittest commands |
| `tests/acceptance/` | Repeatable checks mapped to A-01–A-15; Cloud-only checks retain evidence from WP0/WP8 rather than pretending a local test proves Cloud behavior; package marker allows dotted unittest commands |
| `docs/architecture/` | Data contracts, state transitions, threat model, Cloud operations and recovery procedures |
| `docs/operator/` | Setup, project onboarding, run launch/resume, quota pause, incident and release instructions |
| `docs/operations/` | WP0 Cloud capability evidence and subsequent approved platform decisions |
| `README.md`, `LICENSE`, `CONTRIBUTING.md` | Public factory purpose, quick start, Apache-2.0 terms and safe contribution process |

## Component interfaces and CLI boundary

JSON/JSONL schemas are the durable interchange contract; the Python dataclasses in `contracts.py` are in-process return values and do not introduce a second persisted format. `Diagnostic` is `{path: str, line: int | None, code: str, message: str}`; `RepoRef` is `{repo_id, branch, head_sha, sanitized_remote}`; `RunRef` is `{project_id, run_id, run_dir}`; `WriteResult` is `{changed_paths, expected_sha, resulting_sha}`; `OrgPlan` is `{tree, rationale, requirement_refs}`; `Reservation` is `{parent_id, child_id, unit, amount, status}`; `CommandResult` carries `{ok, exit_code, message, data}`. Other result objects expose the named fields in the interface table and detailed contracts document. Domain errors are `ContractValidationError`, `ConflictError`, `PolicyError`, and `BlockedError`. All paths are explicit `pathlib.Path` values, all mutations require an expected Git SHA or an append-only event, and all errors map to the exit codes defined in Task 1.

| Component | Public interface | Boundary |
|---|---|---|
| `cli.py` | `main(argv: Sequence[str] | None = None) -> int` | Parse args, call exactly one `commands.py` use case, render its result, map typed errors to stable exit codes. No policy decisions, agent spawning, background work, HTTP, credentials or implicit writes. |
| `commands.py` | `initialize_run(factory_root, product_root, project_id, spec_path, privacy_class, expected_factory_sha) -> RunRef`; `validate_run(run_dir) -> ValidationReport`; `show_status(run_dir) -> RunSnapshot`; `checkpoint_run(run_dir, next_action, expected_factory_sha, expected_product_sha) -> WriteResult`; `reconcile_run(run_dir, expected_factory_sha, expected_product_sha, apply=False) -> ReconcileReport`; `build_report(run_dir, output_format) -> Report` | Synchronous command use cases only. `reconcile_run` inspects refs and appends a reconciliation event to the factory repo when `apply=True`; it never edits product files, opens PRs, starts Cloud tasks or invokes models. |
| `validation.py` | `validate_record(record, schema_name) -> list[Diagnostic]`; `validate_json(path, schema_name) -> list[Diagnostic]`; `validate_jsonl(path, schema_name) -> list[Diagnostic]`; `validate_run(run_dir) -> ValidationReport` | Pure validation; no file writes. Diagnostics are stable `{path, line, code, message}` records. |
| `ids.py` | `new_id(kind, clock, entropy) -> str`; `validate_id(value, kind) -> bool` | Stable typed IDs with injectable test sources; uniqueness is never based on wall-clock time alone. |
| `run_store.py` | `create_run(factory_root, product_root, project_id, spec_path, privacy_class, expected_factory_sha) -> RunRef`; `load_run(run_dir) -> RunSnapshot`; `append_event(run_dir, event, expected_factory_sha) -> WriteResult` | Writes only factory records; captures product refs read-only. Atomic file replacement inside one repository is allowed only after SHA check; Git remains the durable history. |
| `state.py` | `validate_run_transition(current, target, evidence) -> None`; `validate_work_transition(current, target, evidence) -> None` | Pure transition rules; raises `PolicyError` for illegal moves. The authorized PO/parent supplies the desired transition; this module does not choose it. |
| `organization.py` | `select_org(requirements, complexity, dependencies, shared_risk, budget, cloud_capabilities) -> OrgPlan` | Pure recommendation with rationale; PO role makes and journals the organization decision. Never spawns agents. |
| `assignments.py` | `split_assignment(parent, child_specs) -> list[Assignment]`; `validate_assignment(parent, child) -> list[Diagnostic]`; `validate_requirement_closure(parent, children) -> list[Diagnostic]` | Pure contract and tree validation. PO/Chief/Planner authors assignment and asks Codex to spawn the corresponding native subagent. |
| `ledger.py` | `reserve(parent_budget, child_request) -> Reservation`; `can_spawn(open_threads, requested_threads, cloud_capabilities, remaining_allowance) -> ReservationDecision`; `append_usage_event(run_dir, event, expected_factory_sha) -> WriteResult` | Records estimates/limits and rejects unauthorized fan-out. It cannot observe or hard-cap subscription consumption except where account evidence is entered by an authorized role. |
| `gitops.py` | `inspect_repo(repo_path) -> RepoRef`; `compare_expected_sha(repo_path, expected_sha) -> bool`; `reconcile_saga(run_dir, expected_factory_sha, expected_product_sha, apply=False) -> ReconcileReport` | Read-only Git inspection plus optional factory-side reconciliation event. Shell out only with argument arrays and `shell=False`; sanitize remotes before journaling. No product mutation, `push --force`, GitHub API or PR creation. |
| `acceptance.py` | `validate_local_acceptance(assignment, evidence) -> list[Diagnostic]`; `validate_product_acceptance(manifest, evidence) -> list[Diagnostic]` | Checks criteria completeness and reviewer authority; the role agent makes and records the acceptance decision. |
| `privacy.py`, `redaction.py` | `authorize_action(action, actor, authority) -> AuthorizationDecision`; `prepare_public_journal(text, privacy_class) -> RedactionReport` | Denies unapproved protected actions and returns redaction findings; a human still approves publication of private/uncertain content. |
| `pause.py` | `checkpoint_for_pause(run_dir, reason, next_action, expected_factory_sha, expected_product_sha) -> WriteResult`; `validate_resume(run_snapshot, fresh_usage_evidence, repo_refs) -> None` | Records a safe checkpoint and checks resume preconditions; it cannot resume a Cloud task or override a human approval. |
| `experiments.py`, `metrics.py` | `validate_experiment_transition(experiment, target, authority) -> None`; `calculate_report(run_snapshot, events) -> Report` | Validate authorized experiment lifecycle and calculate summaries from persisted evidence; never changes role instructions or autonomously adopts an experiment. `commands.build_report` formats this result. |

The eight schema contracts have these required top-level fields and invariants; the individual JSON Schema files must encode these rules with `additionalProperties: false` except for explicitly named extension maps:

| Record | Required fields / checks |
|---|---|
| Manifest | `schema_version`, `project_id`, `run_id`, `privacy_class`, `specification_ref` (path and SHA), `factory_repo`, `product_repo`, `org_tree`, `pinned_versions`, `state`, `requirement_coverage`, `next_action`; start refs are immutable and transitions are checked against the FSM. |
| Assignment | `schema_version`, `assignment_id`, `run_id`, `parent_id`, `task_id`, `requirement_ids`, `objective`, `context_refs`, `allowed_paths`, `dependencies`, `deliverables`, `acceptance_checks`, `authority_snapshot`, `resource_reservation`, `retry_cap`, `stop_conditions`, `escalation_conditions`; IDs/references must resolve and dependencies must be acyclic. |
| Event | `schema_version`, `event_id`, `run_id`, `event_type`, `actor`, `timestamp`, `payload`, `source_refs`; duplicate IDs are rejected and JSONL is append-only. |
| Decision | `schema_version`, `decision_id`, `run_id`, `decision_type`, `authority`, `rationale`, `evidence_refs`, `timestamp`; corrections name the superseded decision/event and append a replacement without editing history. |
| Resource event | `schema_version`, `event_id`, `run_id`, `parent_id`, `child_id`, `resource`, `unit`, `window`, `estimate`, `reserved`, `observed`, `status`, `source`, `reliability`, `stop_decision`; unknown values are explicit `null` with `status: "unknown"`. |
| Sync operation | `schema_version`, `operation_id`, `run_id`, `idempotency_key`, `operation_type`, `factory_expected_sha`, `product_expected_sha`, `intent`, `status`, `product_result_ref`, `factory_result_ref`, `reconciliation`; intent precedes external-repository mutation, and an ambiguous result is `BLOCKED`. |
| Acceptance | `schema_version`, `acceptance_id`, `run_id`, `assignment_id` or product scope, `acceptance_level`, `criteria_refs`, `evidence_refs`, `reviewer_id`, `decision`, `timestamp`; local acceptance is by direct parent and integrated product acceptance by PO only. |
| Experiment | `schema_version`, `experiment_id`, `scope`, `state`, `baseline_sha`, `candidate_sha`, `authorization_ref`, `comparison_cohort`, `success_metrics`, `guardrails`, `sample_limitations`, `approver`, `rollback_trigger`, `result_refs`; authorization precedes evaluation/adoption and versions are immutable. |

`pyproject.toml` must include `src/ai_factory/schemas/*.schema.json` as package data so the installed CLI uses the same contracts as the repository. Contract docs define enum values, reference resolution, redaction fields and transition evidence; schema tests cover each record type and reject unknown fields.

The Python CLI is justified as a small deterministic helper for validating shared JSON contracts, creating/checkpointing factory-side records, checking Git refs and producing reports. It prevents each role from reimplementing safety-sensitive parsing and SHA checks. It is not the factory runtime: it has no agent/provider client, spawn/dispatch function, scheduler, queue consumer, background process, persistent service, hidden state store or authority to choose work, organization, model, acceptance or next action. It performs one finite command and exits. The Codex root PO and native Chief/Planner/Worker subagents own workflow orchestration: choose/split work, spawn and wait for child threads, inspect output, approve work and decide next actions. If Python/the CLI is unavailable, the task is blocked for restoration of the documented Cloud environment/tooling or explicit human decision; the implementation does not gain a parallel orchestration path.

## Dependency order

```text
WP0 Cloud capability gate
  ├── WP1 schemas and run contracts
  │     └── WP2 Git persistence and two-repository saga
  ├── WP3 native role/subagent workflows (WP0–WP2)
  ├── WP4 budget ledger, pause/resume, retry caps (WP1–WP3)
  ├── WP5 assignment acceptance, escalation, coverage (WP1–WP4)
  ├── WP6 authorized self-correction (WP1, WP3, WP5)
  ├── WP7 security/privacy hardening (WP0–WP6)
  └── WP8 E2E acceptance and release (WP0–WP7)
```

Follow the approved roadmap dependencies: WP1 follows WP0; WP2 follows WP1; WP3 follows WP0–WP2; WP4 follows WP1–WP3; WP5 follows WP1–WP4; WP6 follows WP1/WP3/WP5; WP7 follows WP0–WP6; WP8 follows WP0–WP7. Independent files within a work package may be developed in parallel when their contracts are fixed. Integrate in dependency order and serialize shared-manifest writes. WP8 is a release gate, not a scope-reduction stage.

## Implementation tasks

### Task 0: Close the Codex Cloud WP0 capability gate

**Files:**
- Create: `docs/operations/wp0-codex-cloud-evidence.md`
- Update: `docs/superpowers/specs/2026-10-18-ai-factory-design.md` only if the user approves an architecture decision after reviewing evidence

- [ ] In the intended ChatGPT account, open one published Cloud environment with the factory repository and a disposable product test repository. Record account/plan eligibility, environment/repo selection, setup, network policy and usage screen evidence.
- [ ] In a single root task, spawn a child, ask that child to spawn a grandchild, and record the deepest successful tree. Attempt the required `PO → Chief → Planner → Worker` path.
- [ ] Raise the nested tree to the required acceptance topology `PO → Chief → 2 Planners → 1 Worker per Planner`; record which threads remain open and the largest simultaneous child count accepted. Test up to six child threads, then one beyond six if permitted by the environment, and record rejection/behavior without assuming a published example is a limit.
- [ ] Compare root model/reasoning with inherited child settings, then test an explicit different model/reasoning on a child and a nested child. Record available choices and whether actual model settings are observable.
- [ ] Verify whether the Cloud task loads root `AGENTS.md`, `.codex/config.toml`, `.codex/agents/chief.toml`, `.codex/agents/planner.toml`, `.codex/agents/worker.toml` and the five skill files under `.agents/skills/`; separately test generic native spawn with role prompts referencing `factory/roles/po.md`, `factory/roles/chief.md`, `factory/roles/planner.md` and `factory/roles/worker.md`. Record model overrides and tool/permission inheritance. A missing custom-profile loader does not authorize a task relay: use the generic native spawn protocol in the same root task if WP0 confirms it supports the required model/role behavior; otherwise record a blocker.
- [ ] Create a disposable change in each connected repository and verify separate commits/PRs, identity, branch behavior and recoverability. Inspect plan usage/reset visibility and whether subagent usage is distinguishable; record `unknown` where it is not.
- [ ] Write an evidence table with timestamp, account/plan class (no personal identifiers), environment/repo refs, action, result, screenshot/task/thread reference, conclusion and affected P/A IDs. Do not include credentials, private source or sensitive account details.
- [ ] **Gate:** if depth, concurrency or model controls do not support the required one-task topology, stop and present an explicit blocker/options to the user. Continue only after an approved architecture decision. No code is written before this gate passes.

### Task 1: Bootstrap the installable factory package and local test baseline

**Files:**
- Create: `pyproject.toml`, `src/ai_factory/__init__.py`, `src/ai_factory/__main__.py`, `src/ai_factory/cli.py`, `src/ai_factory/commands.py`, `src/ai_factory/contracts.py`
- Create: `tests/__init__.py`, `tests/unit/__init__.py`, `tests/integration/__init__.py`, `tests/acceptance/__init__.py`, `tests/unit/test_cli.py`, `.gitignore`, `LICENSE`, `CONTRIBUTING.md`
- Modify: `README.md`

- [ ] Add a failing test `test_cli_help_lists_commands` asserting `python -m ai_factory --help` exits 0 and lists `init`, `validate`, `status`, `checkpoint`, `reconcile`, and `report`.
- [ ] Run `python -m unittest tests.unit.test_cli -v`; confirm it fails because the module/entry point is absent.
- [ ] Add a minimal Python 3.12 package with a `jsonschema>=4.23,<5` dependency and a console script `ai-factory = ai_factory.cli:main`. Define command names and stable exit codes in `cli.py`: `0` success, `2` invalid input/contract, `3` conflict or blocked reconciliation, `4` provider/environment prerequisite unavailable.
- [ ] Implement these exact CLI forms and only argument parsing/help in this task: `ai-factory init --factory PATH --product PATH --project-id ID --spec PATH --privacy-class public|private --expected-factory-sha SHA`, `ai-factory validate RUN_DIR`, `ai-factory status RUN_DIR`, `ai-factory checkpoint RUN_DIR --next-action TEXT --expected-factory-sha SHA --expected-product-sha SHA`, `ai-factory reconcile RUN_DIR [--dry-run|--apply]` (dry-run is default; `--apply` requires `--expected-factory-sha SHA --expected-product-sha SHA`), and `ai-factory report RUN_DIR --format text|json`. Add global `--json` only where a command has structured output. Do not add cloud API clients, credentials, database access or task-start automation.
- [ ] Install the package in the active development environment with `python -m pip install -e .`; expect successful editable installation and `ai-factory --help` to resolve the package entry point.
- [ ] Rerun `python -m unittest tests.unit.test_cli -v`; expect PASS. Then run `python -m unittest discover -s tests -v`; expect all current tests PASS.
- [ ] Document setup and the fact that Cloud task creation/resumption is done by a human in ChatGPT UI. Add Apache-2.0 license text and contribution rules prohibiting secrets/product source in factory fixtures.
- [ ] Commit as `chore: bootstrap AI Factory package`.

### Task 2: Define versioned JSON contracts and strict validation

**Files:**
- Create: `src/ai_factory/schemas/manifest.schema.json`, `src/ai_factory/schemas/assignment.schema.json`, `src/ai_factory/schemas/event.schema.json`, `src/ai_factory/schemas/decision.schema.json`, `src/ai_factory/schemas/resource-event.schema.json`, `src/ai_factory/schemas/sync-operation.schema.json`, `src/ai_factory/schemas/acceptance.schema.json`, `src/ai_factory/schemas/experiment.schema.json`
- Create: `src/ai_factory/validation.py`, `tests/unit/test_validation.py`, `docs/architecture/data-contracts.md`
- Modify: `pyproject.toml`

- [ ] Write tests `test_valid_manifest_passes`, `test_unknown_manifest_field_fails`, `test_valid_assignment_requires_stop_and_escalation`, `test_valid_event_and_decision_records_pass`, `test_resource_unknown_usage_is_not_zero`, `test_sync_operation_requires_intent_before_result`, `test_acceptance_requires_correct_reviewer_authority`, `test_experiment_requires_authorization_before_adoption`, `test_missing_cross_record_reference_fails`, and `test_malformed_jsonl_reports_line_number`. Use `tempfile.TemporaryDirectory` so tests do not touch production records.
- [ ] Run `python -m unittest tests.unit.test_validation -v`; confirm the missing schemas/validator fail.
- [ ] Define JSON Schema Draft 2020-12 contracts. Every record has `schema_version`; IDs are immutable strings; unknown or unavailable measurements use `null` plus `status: "unknown"` and `source: "unavailable"`, never numeric zero as a substitute. Schemas reject unknown fields by default and only permit specifically declared extension maps.
- [ ] Encode the field-level contract table above in all eight schemas, document enums/reference-resolution rules in `docs/architecture/data-contracts.md`, and include the schema files in the built package via `pyproject.toml` package-data configuration.
- [ ] Define manifest fields for project/run IDs, privacy class, specification SHA, factory/product repo IDs and start refs, org tree, pinned role/policy/model versions, state, requirement coverage and next recovery action. Define assignments with parent/task/requirements, objective, allowed context/path refs, dependencies, deliverables, acceptance checks, authority snapshot, resource reservation, retry cap and stop/escalation criteria.
- [ ] Define append-only event, decision, resource, sync intent/result, acceptance and experiment contracts from the approved system design. Reject unknown fields by default; allow only documented extension fields.
- [ ] Implement `validate_record(record, schema_name)`, `validate_json(path, schema_name)`, `validate_jsonl(path, schema_name)` and `validate_run(run_dir)` returning structured diagnostics `{path, line, code, message}` without modifying inputs. Implement cross-record checks for duplicate event IDs, invalid parent IDs and missing requirement refs.
- [ ] Rerun the focused validator tests and `python -m unittest discover -s tests -v`; expect PASS. Commit as `feat: define versioned factory data contracts`.

### Task 3: Implement run and work-item lifecycle plus stable identifiers

**Files:**
- Create: `src/ai_factory/ids.py`, `src/ai_factory/state.py`, `tests/unit/test_ids.py`, `tests/unit/test_state.py`, `docs/architecture/run-lifecycle.md`

- [ ] Write tests `test_ids_are_unique_and_validate`, `test_run_cannot_activate_without_refs_org_and_budget`, `test_paused_run_resumes_only_after_reconciliation`, `test_review_requires_all_required_work_accepted`, `test_only_po_accepts_integrated_product`, and `test_terminal_failure_requires_reason_actor_and_decision`.
- [ ] Run `python -m unittest tests.unit.test_ids tests.unit.test_state -v`; confirm tests fail before implementation.
- [ ] Implement stable ID functions for project, run, task, agent, event and operation records. Accept an injectable clock/random source in tests; never use timestamps alone as uniqueness.
- [ ] Encode the approved FSM: run `PREPARING → ACTIVE ↔ PAUSED → INTEGRATION_REVIEW → ACCEPTED`, with `BLOCKED` recoverable; `CANCELLED`/`FAILED` require explicit reason, actor and decision. Encode work item `BACKLOG → READY → ASSIGNED → RUNNING → REVIEW → ACCEPTED`, optional `CHANGES_REQUESTED`, and bounded retry/pause/block states.
- [ ] Make illegal transitions return a structured error and never rewrite earlier events. A provider/session `completed` event must not mark a product work item accepted.
- [ ] Rerun focused and full unit tests; commit as `feat: enforce run and work-item state transitions`.

### Task 4: Build Git-backed run initialization, checkpoint and recovery

**Files:**
- Create: `src/ai_factory/run_store.py`, `src/ai_factory/gitops.py`, `tests/integration/test_run_recovery.py`, `tests/integration/helpers.py`, `docs/architecture/recovery.md`
- Create: `factory/templates/manifest.json`, `factory/templates/assignment.json`, `factory/templates/decision.json`

- [ ] Write integration tests with two temporary Git repositories: `test_init_creates_separate_factory_run_records`, `test_checkpoint_pins_both_repo_refs`, `test_recovery_reconstructs_next_action_from_git_only`, `test_expected_sha_mismatch_blocks_write`, and `test_repeated_operation_id_is_idempotent`.
- [ ] Run `python -m unittest tests.integration.test_run_recovery -v`; confirm tests fail because run store/git operations are missing.
- [ ] Implement `init_run(factory_path, product_path, project_id, spec_path, privacy_class, expected_factory_sha)` to compare the factory HEAD before writing, capture both repository IDs/HEAD refs, hash the spec, create unique run directories under `production/active/<project>/<run>/`, render templates and validate the initial manifest before `PREPARING` can become `ACTIVE`.
- [ ] Implement checkpoint writing as a new event/manifest version with next action and child assignment/thread status. Compare both current refs to the supplied expected factory/product SHAs before pinning them and compare the factory SHA again before writing; do not force-push or silently overwrite a competing update.
- [ ] Implement recovery loading from committed Git files only, reconstructing state, decisions, pinned instructions/models, repo refs, open intents and next action. Do not require the old local workspace or Cloud VM.
- [ ] Rerun recovery integration tests and full suite; expect PASS. Commit as `feat: persist and recover factory runs from Git`.

### Task 5: Implement repository saga and reconciliation for independent repos

**Files:**
- Modify: `src/ai_factory/gitops.py`, `src/ai_factory/cli.py`
- Create: `tests/integration/test_cross_repo_saga.py`, `docs/architecture/two-repo-saga.md`

- [ ] Write tests `test_saga_writes_factory_intent_before_product_change`, `test_partial_product_success_reconciles_without_duplicate_commit`, `test_changed_base_sha_creates_conflict`, `test_ambiguous_side_effect_enters_blocked`, and `test_no_force_push_is_used` using local bare/working Git repositories.
- [ ] Run `python -m unittest tests.integration.test_cross_repo_saga -v`; confirm the intended failure injection points are not handled.
- [ ] Implement operation IDs/idempotency keys, expected SHAs, factory intent-before-write, product commit/PR reference recording, factory result recording and final reconciliation. The native Codex role performs the assigned product change and any connected-GitHub PR action; the CLI only inspects refs and records/reconciles factory-side evidence. No API or separate task relay is introduced.
- [ ] Add `ai-factory reconcile RUN_DIR` with explicit `--dry-run` default. It lists unresolved intents and proposed safe next actions; applying a write requires `--apply --expected-factory-sha SHA --expected-product-sha SHA` and passing expected-SHA checks. Ambiguous outcomes stay `BLOCKED` for PO/human review.
- [ ] Rerun focused saga tests and full suite; confirm interrupted and repeated runs do not duplicate or erase history. Commit as `feat: reconcile factory and product repository writes`.

### Task 6: Implement native Codex role instructions and subagent operating model

**Files:**
- Create: `AGENTS.md`, `.codex/config.toml`, `.codex/agents/chief.toml`, `.codex/agents/planner.toml`, `.codex/agents/worker.toml`
- Create: `factory/roles/po.md`, `factory/roles/chief.md`, `factory/roles/planner.md`, `factory/roles/worker.md`, `factory/policies/authority.md`, `factory/policies/delegation.md`, `factory/policies/acceptance.md`, `factory/policies/protected-actions.md`, `factory/models.json`, `config/factory.json`, `.agents/skills/start-run/SKILL.md`, `.agents/skills/delegate/SKILL.md`, `.agents/skills/review-result/SKILL.md`, `.agents/skills/integrate/SKILL.md`, `.agents/skills/recover-run/SKILL.md`, `tests/acceptance/test_role_instruction_contracts.py`, `docs/operator/native-subagents.md`
- Modify: `factory/templates/assignment.json`

- [ ] Write tests `test_each_role_declares_authority_inputs_outputs_and_escalation`, `test_worker_prompt_forbids_self_acceptance_and_factory_self_change`, `test_po_core_is_human_controlled`, `test_child_assignment_contains_all_required_fields`, `test_no_instruction_routes_child_work_to_separate_cloud_task`, `test_agent_toml_has_required_fields_and_valid_role_file`, `test_project_config_uses_wp0_cap_and_no_provider_override`, and `test_codex_skill_files_have_name_and_description`.
- [ ] Run `python -m unittest tests.acceptance.test_role_instruction_contracts -v`; confirm role files/checks are absent.
- [ ] Define root PO orchestration and optional Chief/Planner/Worker role boundaries from the approved authority matrix. Each instruction specifies allowed purpose/context, outputs, acceptance owner, retry/escalation boundaries and forbidden protected actions.
- [ ] Write `AGENTS.md` as root PO instructions: point to `factory/roles/po.md`, define the root as the only top-level run authority, require native delegation for child assignments, wait on child results, journal thread refs, and prohibit cross-task handoff. Do not duplicate the full role prompt in `AGENTS.md`.
- [ ] Write `.codex/config.toml` with `[agents] enabled = true`, `max_concurrent_threads_per_session` set to the integer recorded as the verified cap in `docs/operations/wp0-codex-cloud-evidence.md`, and `interrupt_message = true`. Leave default subagent model/reasoning unset so unspecified children inherit root settings. Set the thread cap no higher than six and no higher than WP0 proved; if project config is ignored in Cloud, enforce the measured limit in `config/factory.json` and verify the runtime cap before fan-out.
- [ ] Create `.codex/agents/chief.toml`, `.codex/agents/planner.toml`, and `.codex/agents/worker.toml`. Each must contain unique `name`, role-specific `description`, and concise `developer_instructions` that point respectively to `factory/roles/chief.md`, `factory/roles/planner.md` and `factory/roles/worker.md`, plus applicable `.agents/skills` procedures. Include `model` and `model_reasoning_effort` only for role-specific combinations accepted in WP0; omission means inherit. Do not put `sandbox_mode`, `approval_policy`, provider selection or MCP server definitions in these files; those are inherited from root. Add feature reductions only for tools verified unnecessary and honored by Cloud; prompts never stand in for sandbox enforcement.
- [ ] Create `.agents/skills/start-run/SKILL.md`, `.agents/skills/delegate/SKILL.md`, `.agents/skills/review-result/SKILL.md`, `.agents/skills/integrate/SKILL.md` and `.agents/skills/recover-run/SKILL.md` with valid `name` and `description` front matter and exact invocation guidance. Root instructions link to the skills; role agent files name the relevant skill paths. WP0 must establish Cloud skill discovery; if discovery fails, role `developer_instructions` must include the essential task procedure inline or explicitly direct reading the canonical skill file, still inside that subagent.
- [ ] Populate `config/factory.json` with the observed maximum nesting depth, max open child threads, required org topology, model inheritance/override capability, tool/permission inheritance and `wp0_evidence_ref`. Keep secrets out. Populate `factory/models.json` only with models/reasoning levels evidenced as available; include `verified_at`, official source/evidence ref, capabilities and explicit `unknown` values.
- [ ] Update `factory/templates/assignment.json` with `agent_profile_ref`, `role_instruction_sha`, `skill_refs`, requested model/effort, observed model/effort, and parent/subagent thread IDs when visible. A missing runtime thread/model fact is `unknown`, never fabricated.
- [ ] Verify each `.codex/agents/*.toml` parses with Python `tomllib`; verify each skill metadata field and link resolves. Manually review all role instructions against the authority matrix before commit.
- [ ] Run `python -m unittest tests.acceptance.test_role_instruction_contracts -v`; expect all role, config and skill contract tests PASS. If Cloud model override is unavailable but the required topology still passes with inherited settings, record inherited model in assignments and continue; there is no fallback that passes assignments to another Cloud task.
- [ ] Manually review all prompts against P-008–P-024, P-033, P-037–P-040 and P-045–P-048. Commit as `feat: define native Codex production roles`.

### Task 7: Implement dynamic organization, assignment scheduling and acceptance records

**Files:**
- Create: `src/ai_factory/organization.py`, `src/ai_factory/assignments.py`, `src/ai_factory/acceptance.py`, `tests/unit/test_organization.py`, `tests/unit/test_assignments.py`, `tests/unit/test_acceptance.py`, `docs/architecture/organization-and-acceptance.md`

- [ ] Write tests `test_simple_work_uses_po_and_worker_only`, `test_medium_work_adds_planner_only_when_justified`, `test_complex_work_supports_chief_with_multiple_planners`, `test_reorganization_preserves_completed_task_ids`, `test_parent_requirement_coverage_is_union_of_children`, and `test_only_parent_can_accept_child`.
- [ ] Run the focused tests; confirm expected failures for missing organization and acceptance functions.
- [ ] Implement `select_org(complexity, dependencies, shared_risk, available_budget, cloud_capabilities)` returning an explainable tree with PO always present and at least one Worker, optional managers only with recorded reasons, and no tree deeper/larger than WP0 capabilities.
- [ ] Implement requirement-preserving `split_assignment(parent, children)`; reject missing parent requirement IDs, unapproved scope expansion, cyclic dependencies, absent acceptance criteria and children exceeding budget/retry/concurrency allocations.
- [ ] Implement direct-parent local acceptance and PO-only product integration acceptance as separate records. Require immutable source criteria and evidence refs. A passed unit test cannot substitute for missing external criteria.
- [ ] Rerun focused and full tests; commit as `feat: plan hierarchical work and record acceptance`.

### Task 8: Implement append-only journal, resource ledger, concurrency caps and pause/resume

**Files:**
- Create: `src/ai_factory/ledger.py`, `src/ai_factory/pause.py`, `tests/unit/test_ledger.py`, `tests/unit/test_pause.py`, `docs/architecture/resource-management.md`, `docs/operator/quota-and-recovery.md`

- [ ] Write tests `test_journal_appends_superseding_correction`, `test_reservation_cannot_exceed_parent_budget`, `test_unknown_usage_never_counts_as_zero`, `test_spawn_respects_wp0_thread_cap`, `test_exhausted_quota_pauses_run`, and `test_resume_requires_fresh_usage_and_reconciliation`.
- [ ] Run focused tests and confirm they fail before ledger/pause implementation.
- [ ] Implement append-only JSONL event writing with unique event IDs, parent/child reservation paths, resource type/unit/window, estimate/reserved/observed/unknown, source/reliability and stop decision. Corrections append a superseding event; they do not edit prior lines.
- [ ] Implement reservation checks for run/parent/child assignments and `max_open_subagents` from WP0. Enforce `min(cloud_verified_limit, configured_target)`; unknown remaining plan allowance blocks new parallel fan-out and moves the run to `PAUSED` at the next safe boundary.
- [ ] Implement bounded retry accounting and repeated failure fingerprint escalation. Resetting allowance is never automated. Resume requires a checkpoint, reconciled repository refs, explicit continuation and fresh available-usage evidence or a human-approved documented unknown policy.
- [ ] Rerun focused and full tests; commit as `feat: track usage and pause safely at quota boundaries`.

### Task 9: Implement controlled self-correction and experiment evaluation

**Files:**
- Create: `src/ai_factory/experiments.py`, `tests/unit/test_experiments.py`, `tests/acceptance/test_instruction_governance.py`, `docs/architecture/self-correction.md`, `factory/templates/experiment.json`, `experiments/README.md`

- [ ] Write tests `test_observation_cannot_change_instruction`, `test_experiment_requires_parent_authorization`, `test_active_assignment_keeps_pinned_instruction_hash`, `test_new_runs_use_adopted_instruction`, `test_rollback_is_a_new_version`, and `test_cross_product_generalization_requires_comparable_evidence`.
- [ ] Run focused tests and verify all unauthorized prompt mutations are denied.
- [ ] Implement states `OBSERVATION → PROPOSAL → AUTHORIZED_EXPERIMENT → EVALUATION → ADOPTED | REJECTED | ROLLED_BACK`; record baseline/comparison cohort, instruction hashes, sample limitations, success/guardrail metrics, approver and rollback trigger.
- [ ] Apply authority chain: direct parent controls direct-report base prompts; only human changes PO core/immutable policy. Worker cannot analyze factory performance or approve its own instruction. Existing assignment pins do not change after adoption.
- [ ] Implement stratified comparisons by task class/complexity and recording model/tool versions, acceptance, rework, retries, escalation, recovery and known/unknown usage. Do not infer a general factory improvement from one product incident.
- [ ] Rerun focused and acceptance tests; commit as `feat: govern factory instruction experiments`.

### Task 10: Implement privacy, redaction, least privilege and protected-action gates

**Files:**
- Create: `src/ai_factory/redaction.py`, `src/ai_factory/privacy.py`, `tests/unit/test_redaction.py`, `tests/unit/test_privacy.py`, `tests/acceptance/test_protected_actions.py`, `docs/architecture/threat-model.md`, `docs/operator/privacy-and-secrets.md`, `factory/policies/privacy.md`
- Modify: `factory/policies/authority.md`, `factory/policies/protected-actions.md`

- [ ] Write tests `test_public_journal_redacts_secret_patterns`, `test_private_product_source_is_not_copied_to_factory_public_record`, `test_redaction_reports_findings_for_human_review`, `test_repo_content_cannot_elevate_agent_authority`, `test_deploy_purchase_delete_and_policy_change_require_approval`, and `test_cloud_secret_is_referenced_not_logged`.
- [ ] Run focused tests and confirm the risky paths fail closed.
- [ ] Implement deterministic redaction for configured token/key patterns and privacy-class validation. Redaction produces findings and sanitized candidate text; publication of private or uncertain content still requires a human decision.
- [ ] Implement action policy checks for deploy, purchase, destructive actions, branch/security changes, public release of private material, external commitments and factory governance edits. Each denied/approved action appends an event with actor, authority and evidence.
- [ ] Ensure environment secret values are never copied into configuration, manifests, journals, CLI output or test fixtures. Store only approved secret names/domain scopes, never values.
- [ ] Rerun focused tests and the security acceptance suite; commit as `feat: enforce privacy and protected-action boundaries`.

### Task 11: Add requirement coverage, status/report commands and operator documentation

**Files:**
- Create: `src/ai_factory/metrics.py`, `tests/unit/test_metrics.py`, `tests/integration/test_cli_workflow.py`, `docs/operator/setup.md`, `docs/operator/project-onboarding.md`, `docs/operator/run-lifecycle.md`, `docs/operator/incident-response.md`, `docs/architecture/requirement-traceability.md`, `docs/architecture/cloud-environment.md`
- Modify: `src/ai_factory/cli.py`, `README.md`

- [ ] Write tests `test_report_covers_all_spec_requirement_ids`, `test_status_shows_parent_child_state_and_next_action`, `test_report_separates_observed_estimated_and_unknown_usage`, and an end-to-end CLI test for `init → validate → status → checkpoint → report` against temporary repositories.
- [ ] Run `python -m unittest tests.unit.test_metrics tests.integration.test_cli_workflow -v`; confirm the report/metrics use cases are missing.
- [ ] Implement `validate`, `status`, `report`, and `init` CLI commands. `report` emits requirement IDs, acceptance state, child-thread references, commit/PR refs, estimates/reservations/observed/unknown usage, open decisions and next action; output is redacted according to privacy class.
- [ ] Write step-by-step operator docs for publishing a Cloud environment, connecting both repos, running the WP0-verified native subagent topology, starting/resuming the root task in ChatGPT UI, checking allowance, pausing, recovering from Git, handling conflicts and managing secrets. Include no commands that create Cloud tasks or call OpenAI APIs.
- [ ] Add a traceability map from each P-001–P-048 and A-01–A-15 to concrete module/instruction/docs and future test/evidence. Every ID must appear exactly once as a primary mapping, even if evidence is shared.
- [ ] Rerun focused and full test suites; commit as `feat: report run state and document Cloud operations`.

### Task 12: Run multi-product end-to-end acceptance and close v1.0 release gate

**Files:**
- Create: `tests/acceptance/test_a01_multi_product.py`, `tests/acceptance/test_a02_flat_run.py`, `tests/acceptance/test_a03_native_hierarchy.py`, `tests/acceptance/test_a04_recursive_split.py`, `tests/acceptance/test_a05_shared_budget.py`, `tests/acceptance/test_a06_recovery.py`, `tests/acceptance/test_a07_bounded_retry.py`, `tests/acceptance/test_a08_a10_prompt_governance.py`, `tests/acceptance/test_a11_instruction_injection.py`, `tests/acceptance/test_a12_product_acceptance.py`, `tests/acceptance/test_a13_saga_replay.py`, `tests/acceptance/test_a14_git_only_recovery.py`, `tests/acceptance/test_a15_generalization.py`, `tests/fixtures/valid-run/manifest.json`, `tests/fixtures/valid-run/events.jsonl`, `tests/fixtures/valid-run/assignment.json`
- Create: `docs/release/v1.0-evidence.md`, `docs/operator/release-checklist.md`
- Modify: `README.md`

Acceptance assertions:

| Scenario | Observable pass condition |
|---|---|
| A-01 | One factory repo serves two independent temporary product repos; product IDs, run IDs, specs and privacy settings remain separate. |
| A-02 | A simple run uses PO→Worker, records the org rationale, worker product commit and factual factory journal, then records PO acceptance. |
| A-03 | In one root Cloud task, PO→Chief→at least two Planners→Worker per Planner completes with role/model/thread refs and independent assignment scopes. |
| A-04 | A Planner recursively splits a large assignment; child requirements, dependencies and acceptance evidence close over the parent without loss. |
| A-05 | Two Planners reserve from one parent allowance; no spawn exceeds either the available shared reservation or WP0 concurrency cap. |
| A-06 | Quota exhaustion checkpoints and pauses the run; fresh usage evidence and reconciliation permit same-task continuation without repeating completed work. |
| A-07 | A Worker failure/retry produces bounded attempts and a factual journal; measured, estimated and unknown usage remain distinct. |
| A-08 | A Planner identifies a defect in its own task-creation instruction and escalates a proposal to Chief; the Planner cannot approve its own change. |
| A-09 | A parent authorizes a direct-report instruction update; the Git version/authority chain is recorded, a later run uses it and an active assignment remains pinned. |
| A-10 | A prompt change is evaluated against a comparable baseline and guardrails; adoption or rollback is evidence-based, with rollback recorded as a new version. |
| A-11 | Product prompt injection cannot change authority, protected constraints or factory rules; only allowed product-local instruction paths may change. |
| A-12 | Integrated product acceptance checks external criteria and integration evidence; local tests alone cannot mark the product accepted. |
| A-13 | Injected failures and concurrent SHA changes at each saga boundary reconcile without duplicate side effects or rewritten history; ambiguity becomes BLOCKED. |
| A-14 | A clean workspace reconstructs run state, decisions, model/prompt pins, repo refs and next action from committed Git files alone. |
| A-15 | Product-only corrections stay in the product repo; any proposed reusable factory change has comparable evidence, correct authority and factory-repo provenance. |

The local harness may exercise deterministic contract/state/recovery assertions for these scenarios, but rows that require Cloud behavior pass only with account/runtime evidence from WP0 or WP8.

- [ ] Use `tests/integration/helpers.py` to construct two independent temporary sample product repositories with different specifications/privacy classes. Fixtures contain synthetic code/data only and are removed with the temporary directories.
- [ ] Run `python -m unittest discover -s tests/acceptance -v`; expect all local acceptance checks PASS, with each test module mapping its named A-ID(s) to an evidence record. Cloud-only conditions remain pending until the next step.
- [ ] Execute A-01–A-15 Cloud-dependent scenarios in the intended account using a single root Cloud task and native Codex subagents. Capture agent/thread IDs, accepted nesting/concurrency, model choice/inheritance and plan-usage evidence. Local tests cannot mark Cloud-only behaviors passed.
- [ ] Execute A-13 failure injection at every two-repository saga boundary and A-14 from a clean workspace using only GitHub factory/product repositories. Verify no duplicate commit, hidden rewrite, repeated side effect or dependency on prior VM.
- [ ] Run `python -m unittest discover -s tests -v`; expect all local tests PASS. Run `python -m ai_factory validate tests/fixtures/valid-run`; expect `VALID` and exit 0. Run `python -m ai_factory report tests/fixtures/valid-run --format json`; expect valid JSON and no secret canary.
- [ ] Verify all P-001–P-048 mappings, all A-01–A-15 evidence records, no unresolved critical/high security finding, no open WP0 blocker, approved license/docs, clean repository state and independent reviewer sign-off. If any gate fails, record it and stop; do not label v1.0 accepted.
- [ ] Commit release evidence as `docs: record AI Factory v1.0 acceptance evidence` after the independent review approves the release report.

## Final traceability coverage

| Requirements | Primary implementation tasks |
|---|---|
| P-001–P-007 (repository separation, access, IDs, saga, privacy/public reuse) | 0, 2, 4, 5, 9, 11, 12 |
| P-008–P-012 (instruction layering, immutable requirements, finding ownership) | 2, 6, 9, 10, 11 |
| P-013–P-020 (roles, dynamic hierarchy, models, delegation contracts) | 0, 3, 6, 7 |
| P-021–P-025 (worker boundaries, acceptance, escalation, retry limits) | 3, 6, 7, 8, 9 |
| P-026–P-031 (shared resources, limits, pause/recovery, platform gates) | 0, 4, 5, 8, 11 |
| P-032–P-036 (journals, events, checkpoint/state recovery) | 2, 3, 4, 5, 8, 11 |
| P-037–P-043 (self-analysis boundary, prompt governance, experiments) | 6, 9, 11 |
| P-044–P-048 (Cloud-only default, configuration, security/privacy/protected actions) | 0, 1, 6, 10, 11, 12 |
| A-01–A-06 | 0, 4, 5, 7, 8, 12 |
| A-07–A-10 | 6, 8, 9, 12 |
| A-11–A-15 | 5, 7, 9, 10, 11, 12 |

## Completion definition

The implementation is complete only when WP0 is recorded and accepted, WP1–WP8 exit checks pass, the factory can be reused with at least two separate products, the required native subagent tree is evidenced inside one Codex Cloud root task, all P-001–P-048 are implemented and traced, A-01–A-15 have retained evidence, the independent review has no unresolved release-blocking finding, and the v1.0 release report is approved. Finishing individual tasks or passing local unit tests alone does not constitute v1.0 acceptance.
