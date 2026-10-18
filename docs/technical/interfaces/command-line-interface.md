# Python package and CLI contract

## Purpose and boundary

The Python package is a deterministic helper used by human operators and Codex roles for shared contract validation, safe factory-side record operations, Git ref inspection and reporting. It removes duplicated parsing/SHA checks from role prompts. It does not orchestrate production.

Every invocation parses one finite command, executes synchronously and exits. No daemon, scheduler, persistent in-memory process, queue, work selector, agent/provider client, HTTP/API integration, model selection, automatic acceptance, task creation or task resumption is allowed. The native Codex root PO and its subagents choose/decompose work, spawn/wait on children, inspect results and decide next steps. If the helper cannot run, work blocks for documented tooling restoration or an explicit human decision; no second orchestration implementation is introduced.

## Installation contract

- Python 3.12 runtime for the initial implementation.
- Installable package exposes `ai-factory` and `python -m ai_factory`.
- Runtime dependency is `jsonschema>=4.23,<5` for Draft 2020-12 validation; Git CLI is an environment executable and all invocations use argument arrays with `shell=False`.
- Schemas are package data and available offline to the installed package.
- No secrets/API credentials are accepted as command-line arguments or environment values used for OpenAI API access.

## Commands

| Command | Required inputs | Behavior | Mutation |
|---|---|---|---|
| `ai-factory init --factory PATH --product PATH --project-id ID --spec PATH --privacy-class public\|private --expected-factory-sha SHA` | Factory/product local checkouts, approved spec, privacy class, current factory HEAD | Validate repos/spec/permissions, capture immutable source SHA and both repo starts, create `PREPARING` run and validate resulting records before activation | Factory repo only; compare expected SHA first. Does not write product repo. |
| `ai-factory validate RUN_DIR` | Run directory | Validate schemas, journals, cross-record references, FSM and current repository consistency; output diagnostics | None |
| `ai-factory status RUN_DIR` | Run directory | Show state, role/task tree, acceptance, open operations, resources by class, current refs and next action | None |
| `ai-factory checkpoint RUN_DIR --next-action TEXT --expected-factory-sha SHA --expected-product-sha SHA` | Run path, explicit next safe action and both current expected SHAs | Validate current refs, record checkpoint/child state/open operation refs and update manifest/event | Factory repo only after compare; product repo read-only |
| `ai-factory reconcile RUN_DIR [--dry-run\|--apply]` | Run path; dry-run default | Inspect unresolved saga operations and both repos, report detected state and safe proposed next step | Dry-run none. Apply requires both `--expected-factory-sha SHA` and `--expected-product-sha SHA`; can append reconciliation evidence to factory only. |
| `ai-factory report RUN_DIR --format text\|json` | Run path and output format | Produce requirement/acceptance/status/resource/decision/recovery summary, privacy filtered | None |

The written command synopsis is the compatibility contract. `init`, `checkpoint` and `reconcile --apply` must refuse stale factory SHA; operations involving a product ref also compare its expected SHA before writing any factory claim about it. `--apply` is never implicit. There is no `spawn`, `run`, `schedule`, `cloud`, or `agent` subcommand.

## In-process interfaces

| Module | Public operation | Returns / errors |
|---|---|---|
| `cli.py` | `main(argv: Sequence[str] | None = None) -> int` | Stable process exit code; renders exactly one use case result. |
| `commands.py` | `initialize_run(factory_root, product_root, project_id, spec_path, privacy_class, expected_factory_sha) -> RunRef` | Validates inputs, delegates persistence operation, returns IDs/path. |
|  | `validate_run(run_dir) -> ValidationReport`; `show_status(run_dir) -> RunSnapshot` | Read-only report and snapshot. |
|  | `checkpoint_run(run_dir, next_action, expected_factory_sha, expected_product_sha) -> WriteResult` | Factory-side write after both refs validated. |
|  | `reconcile_run(run_dir, expected_factory_sha, expected_product_sha, apply=False) -> ReconcileReport` | Read-only by default; factory reconciliation record if authorized apply succeeds. |
|  | `build_report(run_dir, output_format) -> Report` | Privacy-filtered, deterministic output. |
| `validation.py` | `validate_record(record, schema_name)`, `validate_json(path, schema_name)`, `validate_jsonl(path, schema_name)`, `validate_run(run_dir)` | Structured `Diagnostic[]`; no mutation. |
| `run_store.py` | `create_run(..., expected_factory_sha)`, `load_run(run_dir)`, `append_event(run_dir, event, expected_factory_sha)` | Factory-only durable record operations; compare-before-write. |
| `gitops.py` | `inspect_repo(path)`, `compare_expected_sha(path, sha)`, `reconcile_saga(run_dir, expected_factory_sha, expected_product_sha, apply=False)` | Read-only inspection plus optional factory-side reconciliation. Never product mutation or PR creation. |
| `metrics.py` | `calculate_report(run_snapshot, events)` | Summary from validated persisted records; does not change lifecycle state. |

Typed returns and error definitions are in the [implementation guide](../implementation/implementation-guide.md). Persisted interchange data remains JSON/JSONL from [data contracts](data-contracts.md).

## Exit codes and output

| Exit | Meaning | Expected consumer action |
|---:|---|---|
| `0` | Command succeeded; validation has no blocking errors | Continue to the next explicitly authorized action. |
| `2` | Invalid CLI input, malformed record, schema/reference/policy validation error | Correct input or record; do not retry unchanged. |
| `3` | Expected SHA conflict, unresolved/ambiguous reconciliation, or policy-blocked action | Stop writes; inspect and reconcile/escalate. |
| `4` | Required local tool/environment prerequisite unavailable | Restore documented environment or escalate; never switch to API path. |

Machine-readable output has stable top-level `ok`, `command`, `exit_code`, `data` (nullable) and `diagnostics` arrays. Messages are safe for the record's privacy class; raw credentials and private source are never echoed. Human-readable output may summarize but cannot omit blocking diagnostics or change result semantics.

## Failure and safety behavior

- Validate arguments before touching repositories. Resolve paths and confirm expected repository IDs/refs.
- Do not infer a missing expected SHA. Missing precondition for a mutation is an input error.
- Read and hash before write; atomically replace a factory file only after precondition check and record event; never force-push.
- On conflict or ambiguous side effect, preserve input, return code 3 and report `BLOCKED`/reconciliation-needed state.
- Use local Git commands only; do not call GitHub APIs or create PRs. Native Codex role actions and repository connection handle product work outside this CLI.
- Redact or withhold sensitive output according to privacy class. Output should make `unknown` explicit.

