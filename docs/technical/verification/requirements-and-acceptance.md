# Requirements traceability and v1.0 acceptance

## Verification status

This file defines how the eventual implementation must demonstrate the approved requirements. This documentation pass performs no runtime acceptance. **No P-requirement or A-scenario is marked implemented or passed.** Cloud-only evidence is still pending WP0. The exact requirement wording lives in [`../../specification.md`](../../specification.md); this matrix points to the technical implementation contract and expected evidence.

Evidence is a committed, privacy-appropriate artifact: schema/CLI output, Git refs, task/thread references when exposed, test result, reviewer decision or account UI evidence. A local test cannot prove Cloud capability; a platform configuration example cannot prove runtime behavior.

## P-001–P-048 traceability

| ID | Technical realization | Required evidence / acceptance |
|---|---|---|
| P-001 | [System architecture](../architecture/system-architecture.md#repository-ownership) | Two independent product repos use one factory configuration; distinct repo/project/run refs. A-01. |
| P-002 | [System architecture](../architecture/system-architecture.md#repository-ownership), [data contracts](../interfaces/data-contracts.md#repository-layout-and-ownership) | Factory contains reusable policy and operational records, no application source. Audit plus A-14. |
| P-003 | [System architecture](../architecture/system-architecture.md#repository-ownership) | Product repo holds external spec, criteria, local instruction and app artifacts; source SHA pinned. A-12. |
| P-004 | [Agent model](../architecture/agent-operating-model.md#assignment-contract), [security](../architecture/security-and-privacy.md#trust-and-authority) | Assignment context/path scope recorded; test outside-scope attempt and review diff. |
| P-005 | [Data contracts](../interfaces/data-contracts.md#common-rules) | Reconstruct stable IDs and repo/instruction/model/artifact refs from manifest and journals. A-14. |
| P-006 | [Saga](../interfaces/cross-repository-saga.md) | Inject partial writes/conflicts and reconcile without duplicate or hidden history rewrite. A-13. |
| P-007 | [Security/privacy](../architecture/security-and-privacy.md#privacy-classes-and-storage-policy) | Public-source review, secret canary and privacy review; private factory path for sensitive metadata. A-01/A-15. |
| P-008 | [System architecture](../architecture/system-architecture.md#components-and-contracts) | Role/policy versions are pinned to each assignment/run. |
| P-009 | [Agent model](../architecture/agent-operating-model.md#authority-and-instruction-precedence) | Only allowlisted local product instructions at pinned SHA affect product work; scoped update. A-11. |
| P-010 | [Agent model](../architecture/agent-operating-model.md#authority-and-instruction-precedence) | Conflict/injection tests preserve platform, human and role authority. A-11. |
| P-011 | [Data contracts](../interfaces/data-contracts.md#run-manifest), [agent model](../architecture/agent-operating-model.md#acceptance-and-escalation) | Criteria SHA stays pinned; unauthorized edit rejected; customer approval recorded for change. A-12. |
| P-012 | [Self-correction](../architecture/resource-management-and-self-correction.md#factory-improvement-lifecycle) | Findings classified product-only/factory-wide and changes land in the correct repo. A-15. |
| P-013 | [Agent model](../architecture/agent-operating-model.md#role-semantics) | Manifest cannot enter `ACTIVE` without PO function and product Worker. A-02. |
| P-014 | [Agent model](../architecture/agent-operating-model.md#role-semantics) | Flat/simple run has no unjustified manager; org decision records complexity/resources. A-02/A-03. |
| P-015 | [Agent model](../architecture/agent-operating-model.md#role-semantics) | Flat, PO→Planner, PO→Chief→multiple Planners, and feasible direct Planner tree; native Cloud tree proves full A-03. |
| P-016 | [Agent model](../architecture/agent-operating-model.md#assignment-immutability-and-reorganization) | Reorganize a live run; stable task IDs, prior decisions and accepted outputs remain linked. |
| P-017 | [Agent model](../architecture/agent-operating-model.md#role-semantics) | Role-specific context/access review; assignment context refs and least-context evidence. |
| P-018 | [Codex Cloud](../platform/codex-cloud.md#documented-behavior), [data contracts](../interfaces/data-contracts.md#assignment) | Catalog has verified source/date; WP0 tests inheritance/overrides; actual model or unknown recorded. |
| P-019 | [Agent model](../architecture/agent-operating-model.md#native-delegation-protocol), [data contracts](../interfaces/data-contracts.md#assignment) | Recursive split keeps parent requirements/dependencies/criteria; Cloud nesting in WP0. A-04. |
| P-020 | [Data contracts](../interfaces/data-contracts.md#assignment) | Missing objective/scope/dependency/artifact/check/budget/stop/escalation field fails validation. A-04/A-05. |
| P-021 | [Agent model](../architecture/agent-operating-model.md#role-semantics) | Worker artifact plus factual journal; Worker cannot self-accept or self-improve factory. A-07/A-08. |
| P-022 | [Agent model](../architecture/agent-operating-model.md#acceptance-and-escalation), [data contracts](../interfaces/data-contracts.md#acceptance-record) | Direct parent accepts local result; only PO accepts integrated product; negative/positive acceptance tests. A-02/A-12. |
| P-023 | [Agent model](../architecture/agent-operating-model.md#acceptance-and-escalation) | Deliberate Planner contract conflict is escalated to PO; resolution is journaled. |
| P-024 | [Agent model](../architecture/agent-operating-model.md#acceptance-and-escalation), [data contracts](../interfaces/data-contracts.md#events-decisions-and-execution-journals) | Escalation includes evidence, actions, risk and decision request; correct parent authority receives it. |
| P-025 | [Resource management](../architecture/resource-management-and-self-correction.md#retry-and-stop-policy) | Exhausted retry/spawn limits stop; repeated fingerprint triggers manager review. |
| P-026 | [Resource management](../architecture/resource-management-and-self-correction.md#reservation-and-fan-out-decision) | Chief or PO allocation and Planner reservations share one run ledger; no child exceeds allocation/WP0 cap. A-05. |
| P-027 | [Data contracts](../interfaces/data-contracts.md#events-decisions-and-execution-journals), [resource management](../architecture/resource-management-and-self-correction.md#usage-classification) | Estimates/reservations/observations remain distinct; non-attributable plan usage is unknown. |
| P-028 | [Codex Cloud](../platform/codex-cloud.md#approved-product-boundary), [production runbook](../operations/production-runbook.md#4-monitor-resources-and-pause) | Account-plan policy and usage/reset evidence; no API spend, purchase or linear reset assumption. |
| P-029 | [State and recovery](../architecture/state-and-recovery.md#state-model) | Exhaustion/window expiry creates recoverable `PAUSED`, with checkpoint. A-06. |
| P-030 | [State and recovery](../architecture/state-and-recovery.md#resume-and-recovery-algorithm) | Same task continues when available; after VM loss Git-only recovery under same run ID. A-06/A-14. |
| P-031 | [Codex Cloud](../platform/codex-cloud.md#wp0-capability-gate) | Every required Cloud feature has native verified path or explicit blocker; no task relay/API fallback. |
| P-032 | [Data contracts](../interfaces/data-contracts.md#events-decisions-and-execution-journals) | Each Worker artifact has a versioned factual factory journal. A-02/A-07. |
| P-033 | [Agent model](../architecture/agent-operating-model.md#native-delegation-protocol), [data contracts](../interfaces/data-contracts.md#events-decisions-and-execution-journals) | Leader journals plans, assignments, acceptance, resources, escalation and prompt decisions. A-03/A-08/A-09. |
| P-034 | [Data contracts](../interfaces/data-contracts.md#events-decisions-and-execution-journals) | Journals capture factual actions/checks/failures/retries/status; unknown usage stays null/unknown. A-07. |
| P-035 | [State and recovery](../architecture/state-and-recovery.md#durable-files-and-event-semantics) | Parallel writers, commits, superseding correction and no silent rewrite verified. A-13/A-14. |
| P-036 | [State and recovery](../architecture/state-and-recovery.md#state-model) | Manifest/state/recovery action agrees with journals and current repo refs after Git-only reconstruction. A-14. |
| P-037 | [Agent model](../architecture/agent-operating-model.md#role-semantics) | Worker prompt/event authorship has no factory-effectiveness analysis; leadership owns interpretation. A-08. |
| P-038 | [Agent model](../architecture/agent-operating-model.md#authority-and-instruction-precedence) | A role cannot approve/change its own base prompt; upward escalation evidence. A-08. |
| P-039 | [Self-correction](../architecture/resource-management-and-self-correction.md#factory-improvement-lifecycle) | Parent-to-direct-report authority tested for each edge; human alone changes PO core. A-09. |
| P-040 | [Agent model](../architecture/agent-operating-model.md#assignment-contract) | Task-level wording can change within assignment authority without changing role-base SHA. |
| P-041 | [Data contracts](../interfaces/data-contracts.md#experiment-record), [self-correction](../architecture/resource-management-and-self-correction.md#factory-improvement-lifecycle) | Prior/new hashes, authorizer, rationale and scope persist; active run stays pinned; new run uses new approved version. A-09. |
| P-042 | [Self-correction](../architecture/resource-management-and-self-correction.md#factory-improvement-lifecycle) | Comparable baseline, guardrails and justified adopt/reject/rollback; rollback is new version. A-10. |
| P-043 | [Self-correction](../architecture/resource-management-and-self-correction.md#factory-improvement-lifecycle) | Cross-product generalization requires comparable evidence and correct repo ownership. A-15. |
| P-044 | [System architecture](../architecture/system-architecture.md#context-and-deployment-boundary), [Codex Cloud](../platform/codex-cloud.md#approved-product-boundary) | Full production behavior in hosted task; no mandatory user workstation/server or orchestrator. WP0/WP8. |
| P-045 | [System architecture](../architecture/system-architecture.md#components-and-contracts) | Policy, instructions, schemas and role behavior are reviewable versioned repo files; no opaque required planner. |
| P-046 | [Security/privacy](../architecture/security-and-privacy.md#trust-and-authority) | Least repo/network/action scope, hostile repo instruction and unauthorized action tests; no privilege elevation. |
| P-047 | [Security/privacy](../architecture/security-and-privacy.md#privacy-classes-and-storage-policy) | Secret canary/redaction scan/human review; no public confidential source/PII/credential; private mode tested. |
| P-048 | [Security/privacy](../architecture/security-and-privacy.md#protected-actions) | Deploy/purchase/destructive/security/publication/governance actions denied without approval and logged. |

## A-01–A-15 end-to-end acceptance

| ID | Scenario | Pass evidence |
|---|---|---|
| A-01 | Reuse factory for two independent products | Separate product specs/configs/IDs/refs; no cross-product data leak. |
| A-02 | Simple PO→Worker run | Org rationale, product commit, factual journal, local/PO acceptance; no unnecessary manager. |
| A-03 | PO→Chief→2 Planners→Workers in one root task | WP0-verified native depth/concurrency; each assignment/thread/model or unknown is traceable; local and PO acceptance recorded. |
| A-04 | Recursive Planner decomposition | Parent-child requirement/dependency/criteria closure and review evidence. |
| A-05 | Shared resource reservations | Two Planners draw from shared parent budget; fan-out stays within current allocation and verified Cloud concurrency. |
| A-06 | Quota pause and resume | Durable checkpoint; same task resumes when available or Git recovery after VM loss; no repeated accepted work/auto-reactivation. |
| A-07 | Worker error/retry | Bounded attempts and factual journal; estimated/reserved/observed/unknown usage distinct. |
| A-08 | Planner identifies own instruction defect | Worker/Planner evidence, upward proposal, Chief decision; no self-approval. |
| A-09 | Authorized base prompt update | Direct-parent authority chain, old/new hash, next run uses it and in-flight run remains pinned. |
| A-10 | Evaluate and rollback/adopt instruction | Comparable cohort, quality/resource metrics, guardrails, justified decision and version history. |
| A-11 | Product instruction conflict/injection | Authorized product rule applies only within scope; immutable guardrail/authority cannot be overridden. |
| A-12 | Incomplete integrated product with passing local tests | PO refuses product acceptance for missing external criterion; after remedy, separate acceptance decision. |
| A-13 | Partial cross-repo operation/conflict | Detect existing side effect, avoid duplicate/overwrite, reconcile or block ambiguity. |
| A-14 | Recovery with no original local workspace/VM | Git-only reconstruction of run, decisions, pins, refs and next action; operator starts recovery task for same run ID. |
| A-15 | Product-only issue with possible general lesson | Product fix and factory proposal in correct repositories; factory adoption has comparable evidence and authority. |

Local tests may cover deterministic logic, schemas, failure injection and reconstruction. Cloud-only rows are passed only with intended-account runtime evidence. Preserve evidence refs and reviewer in `docs/release/v1.0-evidence.md`; no scenario in this design corpus is already passed.

## Verification layers

1. **Schema/unit:** ID rules, FSM, validation diagnostics, privacy decisions, assignment closure, budget/retry policy, experiment governance and metrics.
2. **Local Git integration:** temporary independent repos for run initialization, expected-SHA conflicts, append-only journals, recovery, and each saga interruption boundary.
3. **Role contract:** TOML parse, role/skill link validation, required fields, no cross-task relay, no worker self-acceptance, and authority checks.
4. **Cloud account:** eligibility, environment, multi-repo read/write, native nesting/concurrency/model controls, permission inheritance, secrets/network, usage UI and task/VM recovery. WP0 is prerequisite.
5. **Acceptance:** all A-01–A-15 complete with both local evidence and Cloud evidence where applicable; independent review and privacy/security gates.

## Release criteria

Release v1.0 is not accepted until WP0–WP8 exits pass; P-001–P-048 have one trace path and evidence; A-01–A-15 have retained evidence; at least two products use the factory; cross-repository recovery and Git-only run recovery work; privacy/security review has no unresolved release-blocking critical/high finding; required Cloud topology is evidenced in one root task; the independent reviewer signs the release report. Any unsatisfied Cloud assumption is a blocker/open decision, not a test pass.

## Independent review checklist

An independent reviewer must:

1. Check all 48 P IDs and 15 A IDs against the product spec and this technical corpus.
2. Recompute role authority, parent-child prompt change rights, P-010 precedence and protected actions.
3. Trace every failure boundary of the two-repository saga and recovery from committed Git only.
4. Review Cloud statements for source, date, client surface and runtime evidence; confirm no local config example is misrepresented as Cloud proof.
5. Inspect public/private records for source/PII/secret/ref leaks.
6. Confirm no API billing, task relay, omitted v1 feature or hidden scope reduction was added.
7. Return findings by severity, requirement IDs, evidence, needed decision/mitigation and release impact.

