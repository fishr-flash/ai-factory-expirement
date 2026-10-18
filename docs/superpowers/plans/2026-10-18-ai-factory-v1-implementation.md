# AI Factory v1.0 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Follow the native Codex subagent tree defined by the approved architecture. Do not hand off role assignments through separate Codex Cloud tasks.

**Goal:** Deliver the complete AI Factory v1.0: a reusable GitHub factory that runs product work in Codex Cloud under a ChatGPT subscription, delegates through native Codex subagents, durably records decisions and execution in Git, and improves its production methods through authorized evidence-based experiments.

**Architecture:** The factory is a versioned repository of role instructions, policies, schemas, Python tooling and run records; product code and product acceptance criteria stay in separate product repositories. A human starts or resumes one root Codex Cloud task in the ChatGPT UI; the PO role orchestrates a native Codex subagent tree in that task, while Git files are the durable state and the Python CLI validates and reconciles records. No API key, Agents/Responses API, custom server, database, or manual child-task relay is part of v1.0.

**Tech Stack:** Python 3.12; `jsonschema>=4.23,<5` for Draft 2020-12 validation (the [official docs](https://python-jsonschema.readthedocs.io/en/stable/) document Draft 2020-12 support); JSON for manifests/contracts/decisions; append-only JSONL for journals and ledgers; Markdown for Codex role instructions, policies and runbooks; `unittest` for tests; Git/GitHub for durable state, branches and PRs; Codex Cloud and native subagents through the ChatGPT subscription UI.

---

## Authority, gates, and execution rules

- Product authority: [`docs/specification.md`](../../specification.md), requirements P-001–P-048.
- Approved architecture: [`2026-10-18-ai-factory-design.md`](../specs/2026-10-18-ai-factory-design.md). Acceptance trace: [`2026-10-18-ai-factory-verification.md`](../specs/2026-10-18-ai-factory-verification.md). This plan does not amend either document.
- WP0 is a hard prerequisite for implementation choices that depend on Cloud behavior. It must record the available nested depth, maximum simultaneously open child threads, model/reasoning inheritance and overrides, Cloud-visible agent configuration, parent permission/tool inheritance, actual account entitlement, two-repository writes, environment secrets and plan-usage evidence. Six child threads is the initial target to verify, not an assumed provider guarantee.
- If WP0 cannot run PO→Chief→Planner→Worker nesting or the required concurrency/model setup in one root Cloud task, stop at the gate and return evidence and options for a human architecture decision. Do not implement a separate-Cloud-task handoff, API path, or quiet reduction of P/A acceptance.
- During execution, use TDD for application behavior: write one focused failing test, run that exact test, implement the smallest change, rerun it, then run the full suite. Keep commits small and task-scoped. Commands below are planned future commands; no implementation or test is performed while this plan is reviewed.
- Every Cloud subagent gets a written assignment contract and role instruction reference. Subagents inherit root permissions; path restrictions are enforced through assignment, review, branch policy and behavior checks, not represented as a per-agent OS sandbox.

## Repository and component map

This plan establishes the following implementation layout. The product repository is selected per run and is never nested inside the factory repository as a permanent product-code directory.

| Path | Responsibility |
|---|---|
| `pyproject.toml` | Installable Python package, Python version, runtime dependency and CLI entry point |
| `src/ai_factory/__init__.py` | Package version |
| `src/ai_factory/__main__.py` | `python -m ai_factory` entry point |
| `src/ai_factory/cli.py` | Argument parsing, command dispatch, exit codes, human-readable and JSON output |
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
| `schemas/*.schema.json` | Draft 2020-12 contracts for manifests, assignments, events, decisions, resources, sync operations, acceptances and experiments |
| `config/factory.json` | Defaults, privacy classes, retry/fan-out ceilings and model catalog schema reference; no secrets or API credentials |
| `factory/roles/{po,chief,planner,worker}.md` | Role authority, job boundaries, input/output contract and escalation protocol |
| `factory/policies/*.md` | Immutable guardrails, authority matrix, privacy/redaction, protected actions and instruction precedence |
| `factory/skills/*.md` | Reusable procedures for run startup, delegation, review, integration, recovery and self-correction |
| `factory/models.json` | Versioned model capability catalog with source/verification date and `unknown` handling |
| `factory/templates/*.json` | Valid starter records for runs, assignments, decisions and experiments |
| `production/active/` and `production/archive/` | Git-durable run directories, separated by project/run and privacy class |
| `experiments/` | Authorized prompt/skill/organization experiments and outcome evidence |
| `tests/unit/` | Deterministic Python unit tests using temporary directories and fake Git repositories; package marker allows dotted unittest commands |
| `tests/integration/` | CLI, JSONL, run recovery and two-repository saga integration tests using local temporary Git repos; package marker allows dotted unittest commands |
| `tests/acceptance/` | Repeatable checks mapped to A-01–A-15; Cloud-only checks retain evidence from WP0/WP8 rather than pretending a local test proves Cloud behavior; package marker allows dotted unittest commands |
| `docs/architecture/` | Data contracts, state transitions, threat model, Cloud operations and recovery procedures |
| `docs/operator/` | Setup, project onboarding, run launch/resume, quota pause, incident and release instructions |
| `docs/operations/` | WP0 Cloud capability evidence and subsequent approved platform decisions |
| `README.md`, `LICENSE`, `CONTRIBUTING.md` | Public factory purpose, quick start, Apache-2.0 terms and safe contribution process |

## Dependency order

```text
WP0 Cloud capability gate
  └── WP1 schemas and run contracts
        └── WP2 Git persistence and two-repository saga
              └── WP3 native role/subagent workflows
                    └── WP4 budget ledger, pause/resume, retry caps
                          └── WP5 assignment acceptance, escalation, coverage
                                └── WP6 authorized self-correction
                                      └── WP7 security/privacy hardening
                                            └── WP8 E2E acceptance and release
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
- [ ] Verify whether project-scoped role files/custom agent configuration are loaded in the Cloud task and what permissions, tools, network access and repository access each child inherits. Record that natural-language path scope is not an independent sandbox.
- [ ] Create a disposable change in each connected repository and verify separate commits/PRs, identity, branch behavior and recoverability. Inspect plan usage/reset visibility and whether subagent usage is distinguishable; record `unknown` where it is not.
- [ ] Write an evidence table with timestamp, account/plan class (no personal identifiers), environment/repo refs, action, result, screenshot/task/thread reference, conclusion and affected P/A IDs. Do not include credentials, private source or sensitive account details.
- [ ] **Gate:** if depth, concurrency or model controls do not support the required one-task topology, stop and present an explicit blocker/options to the user. Continue only after an approved architecture decision. No code is written before this gate passes.

### Task 1: Bootstrap the installable factory package and local test baseline

**Files:**
- Create: `pyproject.toml`, `src/ai_factory/__init__.py`, `src/ai_factory/__main__.py`, `src/ai_factory/cli.py`
- Create: `tests/__init__.py`, `tests/unit/__init__.py`, `tests/integration/__init__.py`, `tests/acceptance/__init__.py`, `tests/unit/test_cli.py`, `.gitignore`, `LICENSE`, `CONTRIBUTING.md`
- Modify: `README.md`

- [ ] Add a failing test `test_cli_help_lists_commands` asserting `python -m ai_factory --help` exits 0 and lists `init`, `validate`, `status`, `checkpoint`, `reconcile`, and `report`.
- [ ] Run `python -m unittest tests.unit.test_cli -v`; confirm it fails because the module/entry point is absent.
- [ ] Add a minimal Python 3.12 package with a `jsonschema>=4.23,<5` dependency and a console script `ai-factory = ai_factory.cli:main`. Define command names and stable exit codes in `cli.py`: `0` success, `2` invalid input/contract, `3` conflict or blocked reconciliation, `4` provider/environment prerequisite unavailable.
- [ ] Implement these exact CLI forms and only argument parsing/help in this task: `ai-factory init --factory PATH --product PATH --project-id ID --spec PATH --privacy-class public|private`, `ai-factory validate RUN_DIR`, `ai-factory status RUN_DIR`, `ai-factory checkpoint RUN_DIR --next-action TEXT`, `ai-factory reconcile RUN_DIR [--dry-run|--apply]` (dry-run is default; `--apply` requires `--expected-factory-sha SHA --expected-product-sha SHA`), and `ai-factory report RUN_DIR --format text|json`. Add global `--json` only where a command has structured output. Do not add cloud API clients, credentials, database access or task-start automation.
- [ ] Install the package in the active development environment with `python -m pip install -e .`; expect successful editable installation and `ai-factory --help` to resolve the package entry point.
- [ ] Rerun `python -m unittest tests.unit.test_cli -v`; expect PASS. Then run `python -m unittest discover -s tests -v`; expect all current tests PASS.
- [ ] Document setup and the fact that Cloud task creation/resumption is done by a human in ChatGPT UI. Add Apache-2.0 license text and contribution rules prohibiting secrets/product source in factory fixtures.
- [ ] Commit as `chore: bootstrap AI Factory package`.

### Task 2: Define versioned JSON contracts and strict validation

**Files:**
- Create: `schemas/{manifest,assignment,event,decision,resource-event,sync-operation,acceptance,experiment}.schema.json`
- Create: `src/ai_factory/validation.py`, `tests/unit/test_validation.py`, `docs/architecture/data-contracts.md`
- Modify: `pyproject.toml`

- [ ] Write tests `test_valid_manifest_passes`, `test_unknown_manifest_field_fails`, `test_assignment_requires_stop_and_escalation`, `test_unknown_usage_is_not_zero`, and `test_malformed_jsonl_reports_line_number`. Use `tempfile.TemporaryDirectory` so tests do not touch production records.
- [ ] Run `python -m unittest tests.unit.test_validation -v`; confirm the missing schemas/validator fail.
- [ ] Define JSON Schema Draft 2020-12 contracts. Every record has `schema_version`; IDs are immutable strings; unknown or unavailable measurements use `null` plus `status: "unknown"` and `source: "unavailable"`, never numeric zero as a substitute.
- [ ] Define manifest fields for project/run IDs, privacy class, specification SHA, factory/product repo IDs and start refs, org tree, pinned role/policy/model versions, state, requirement coverage and next recovery action. Define assignments with parent/task/requirements, objective, allowed context/path refs, dependencies, deliverables, acceptance checks, authority snapshot, resource reservation, retry cap and stop/escalation criteria.
- [ ] Define append-only event, decision, resource, sync intent/result, acceptance and experiment contracts from the approved system design. Reject unknown fields by default; allow only documented extension fields.
- [ ] Implement `validate_json(path, schema)` and `validate_jsonl(path, schema)` returning structured diagnostics `{path, line, code, message}` without modifying inputs. Implement cross-record checks for duplicate event IDs, invalid parent IDs and missing requirement refs.
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
- [ ] Implement `init_run(factory_path, product_path, project_id, spec_path, privacy_class)` to capture both repository IDs/HEAD refs, hash the spec, create unique run directories under `production/active/<project>/<run>/`, render templates and validate the initial manifest before `PREPARING` can become `ACTIVE`.
- [ ] Implement checkpoint writing as new event/manifest version with next action and child assignment/thread status. Use compare-before-write against expected Git SHA; do not force-push or silently overwrite a competing update.
- [ ] Implement recovery loading from committed Git files only, reconstructing state, decisions, pinned instructions/models, repo refs, open intents and next action. Do not require the old local workspace or Cloud VM.
- [ ] Rerun recovery integration tests and full suite; expect PASS. Commit as `feat: persist and recover factory runs from Git`.

### Task 5: Implement repository saga and reconciliation for independent repos

**Files:**
- Modify: `src/ai_factory/gitops.py`, `src/ai_factory/cli.py`
- Create: `tests/integration/test_cross_repo_saga.py`, `docs/architecture/two-repo-saga.md`

- [ ] Write tests `test_saga_writes_factory_intent_before_product_change`, `test_partial_product_success_reconciles_without_duplicate_commit`, `test_changed_base_sha_creates_conflict`, `test_ambiguous_side_effect_enters_blocked`, and `test_no_force_push_is_used` using local bare/working Git repositories.
- [ ] Run `python -m unittest tests.integration.test_cross_repo_saga -v`; confirm the intended failure injection points are not handled.
- [ ] Implement operation IDs/idempotency keys, expected SHAs, intent-before-write, product commit/PR reference recording, factory result recording and final reconciliation. Only local git commands are used; Cloud/GitHub task orchestration remains in Codex UI.
- [ ] Add `ai-factory reconcile RUN_DIR` with explicit `--dry-run` default. It lists unresolved intents and proposed safe next actions; applying a write requires `--apply --expected-factory-sha SHA --expected-product-sha SHA` and passing expected-SHA checks. Ambiguous outcomes stay `BLOCKED` for PO/human review.
- [ ] Rerun focused saga tests and full suite; confirm interrupted and repeated runs do not duplicate or erase history. Commit as `feat: reconcile factory and product repository writes`.

### Task 6: Implement native Codex role instructions and subagent operating model

**Files:**
- Create: `AGENTS.md`, `factory/roles/{po,chief,planner,worker}.md`, `factory/policies/{authority,delegation,acceptance,protected-actions}.md`, `factory/skills/{start-run,delegate,review-result,integrate,recover-run}.md`, `factory/templates/assignment.json`
- Create: `tests/acceptance/test_role_instruction_contracts.py`, `docs/operator/native-subagents.md`
- Modify: `factory/models.json`, `config/factory.json`

- [ ] Write document checks `test_each_role_declares_authority_inputs_outputs_and_escalation`, `test_worker_prompt_forbids_self_acceptance_and_factory_self_change`, `test_po_core_is_human_controlled`, `test_child_assignment_contains_all_required_fields`, and `test_no_instruction_routes_child_work_to_separate_cloud_task`.
- [ ] Run `python -m unittest tests.acceptance.test_role_instruction_contracts -v`; confirm role files/checks are absent.
- [ ] Define root PO orchestration and optional Chief/Planner/Worker role boundaries from the approved authority matrix. Each instruction specifies allowed purpose/context, outputs, acceptance owner, retry/escalation boundaries and forbidden protected actions.
- [ ] In `AGENTS.md` and the delegation procedure, instruct Codex to use native subagents for independent assignments in the same root Cloud task, wait for children, persist assignment/thread refs, and let each direct parent accept its children. Record the WP0-confirmed max nested depth/concurrency in `config/factory.json`; cap fan-out at the lower of confirmed Cloud limit or six initial target threads.
- [ ] Model settings inherit from parent by default; explicit role settings are allowed only for models verified available in WP0. Record actual model/effort or `unknown`. Do not promise separate permission/sandbox scope; require assignment path restrictions, diff review and branch protections.
- [ ] Provide a tested fallback only for unavailable model overrides (inherit root model and record that fact). There is no fallback that passes assignments to another Cloud task.
- [ ] Rerun document contract tests. Manually review all prompts against P-008–P-024, P-033, P-037–P-040 and P-045–P-048. Commit as `feat: define native Codex production roles`.

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
- Create: `src/ai_factory/redaction.py`, `src/ai_factory/privacy.py`, `tests/unit/test_redaction.py`, `tests/unit/test_privacy.py`, `tests/acceptance/test_protected_actions.py`, `docs/architecture/threat-model.md`, `docs/operator/privacy-and-secrets.md`
- Modify: `factory/policies/authority.md`, `factory/policies/protected-actions.md`

- [ ] Write tests `test_public_journal_redacts_secret_patterns`, `test_private_product_source_is_not_copied_to_factory_public_record`, `test_redaction_reports_findings_for_human_review`, `test_repo_content_cannot_elevate_agent_authority`, `test_deploy_purchase_delete_and_policy_change_require_approval`, and `test_cloud_secret_is_referenced_not_logged`.
- [ ] Run focused tests and confirm the risky paths fail closed.
- [ ] Implement deterministic redaction for configured token/key patterns and privacy-class validation. Redaction produces findings and sanitized candidate text; publication of private or uncertain content still requires a human decision.
- [ ] Implement action policy checks for deploy, purchase, destructive actions, branch/security changes, public release of private material, external commitments and factory governance edits. Each denied/approved action appends an event with actor, authority and evidence.
- [ ] Ensure environment secret values are never copied into configuration, manifests, journals, CLI output or test fixtures. Store only approved secret names/domain scopes, never values.
- [ ] Rerun focused tests and the security acceptance suite; commit as `feat: enforce privacy and protected-action boundaries`.

### Task 11: Add requirement coverage, status/report commands and operator documentation

**Files:**
- Create: `src/ai_factory/metrics.py`, `tests/unit/test_metrics.py`, `tests/integration/test_cli_workflow.py`, `docs/operator/{setup,project-onboarding,run-lifecycle,incident-response}.md`, `docs/architecture/{requirement-traceability,cloud-environment}.md`
- Modify: `src/ai_factory/cli.py`, `README.md`

- [ ] Write tests `test_report_covers_all_spec_requirement_ids`, `test_status_shows_parent_child_state_and_next_action`, `test_report_separates_observed_estimated_and_unknown_usage`, and an end-to-end CLI test for `init → validate → status → checkpoint → report` against temporary repositories.
- [ ] Run the focused tests and verify they fail before report/metrics implementation.
- [ ] Implement `validate`, `status`, `report`, and `init` CLI commands. `report` emits requirement IDs, acceptance state, child-thread references, commit/PR refs, estimates/reservations/observed/unknown usage, open decisions and next action; output is redacted according to privacy class.
- [ ] Write step-by-step operator docs for publishing a Cloud environment, connecting both repos, running the WP0-verified native subagent topology, starting/resuming the root task in ChatGPT UI, checking allowance, pausing, recovering from Git, handling conflicts and managing secrets. Include no commands that create Cloud tasks or call OpenAI APIs.
- [ ] Add a traceability map from each P-001–P-048 and A-01–A-15 to concrete module/instruction/docs and future test/evidence. Every ID must appear exactly once as a primary mapping, even if evidence is shared.
- [ ] Rerun focused and full test suites; commit as `feat: report run state and document Cloud operations`.

### Task 12: Run multi-product end-to-end acceptance and close v1.0 release gate

**Files:**
- Create: `tests/acceptance/test_a01_multi_product.py`, `tests/acceptance/test_a02_flat_run.py`, `tests/acceptance/test_a03_native_hierarchy.py`, `tests/acceptance/test_a04_recursive_split.py`, `tests/acceptance/test_a05_shared_budget.py`, `tests/acceptance/test_a06_recovery.py`, `tests/acceptance/test_a07_bounded_retry.py`, `tests/acceptance/test_a08_a10_prompt_governance.py`, `tests/acceptance/test_a11_instruction_injection.py`, `tests/acceptance/test_a12_product_acceptance.py`, `tests/acceptance/test_a13_saga_replay.py`, `tests/acceptance/test_a14_git_only_recovery.py`, `tests/acceptance/test_a15_generalization.py`, `tests/fixtures/valid-run/{manifest.json,events.jsonl,assignment.json}`
- Create: `docs/release/v1.0-evidence.md`, `docs/operator/release-checklist.md`
- Modify: `README.md`

- [ ] Implement test fixtures for two independent sample product repositories with different specifications/privacy classes. Fixtures contain synthetic code/data only.
- [ ] Run `python -m unittest discover -s tests/acceptance -v`; expect all local acceptance checks PASS, with each test module mapping its named A-ID(s) to an evidence record. Cloud-only conditions remain pending until the next step.
- [ ] Execute A-01–A-15 Cloud-dependent scenarios in the intended account using a single root Cloud task and native Codex subagents. Capture agent/thread IDs, accepted nesting/concurrency, model choice/inheritance and plan-usage evidence. Local tests cannot mark Cloud-only behaviors passed.
- [ ] Execute A-13 failure injection at every two-repository saga boundary and A-14 from a clean workspace using only GitHub factory/product repositories. Verify no duplicate commit, hidden rewrite, repeated side effect or dependency on prior VM.
- [ ] Run `python -m unittest discover -s tests -v`; expect all local tests PASS. Run `python -m ai_factory validate production/active/demo/run-0001` on a synthetic fixture; expect `VALID` and exit 0. Run `python -m ai_factory report production/active/demo/run-0001 --format json`; expect valid JSON and no secret canary.
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
