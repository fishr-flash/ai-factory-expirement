# Production runbook

## Purpose and authority

This is the human-operated lifecycle for one AI Factory production run. The operator owns ChatGPT account access and task start/resume; PO owns run production decisions; direct parents own local acceptance; the human/customer retains protected approvals and product requirement authority. Do not use this runbook to hand child assignments between Cloud tasks.

Before proceeding, complete [environment setup](environment-setup.md) and confirm the WP0 evidence for the intended environment in [Codex Cloud platform contract](../platform/codex-cloud.md).

## 1. Intake and authorize

1. Identify product/repository and customer. Confirm the external product specification, acceptance criteria, product constraints and privacy class.
2. Get explicit approval for the source criteria and any consequential scope/launch action. Never ask PO or Worker to infer missing external acceptance criteria.
3. Confirm factory repo visibility is suitable for run metadata; choose private factory where even identifiers/refs could reveal sensitive product information.
4. Record project ID; run initialization creates a unique run ID. Select current factory/product branches and capture full start SHAs. Confirm expected factory SHA for the safe initializer.
5. Identify allowlisted product-local instructions and pin their SHA. Record policy/role/skill/model catalog versions.
6. Check Cloud allowance/usage and current environment. If unknown headroom blocks fan-out under policy, start only the authorized nonparallel intake/checkpoint work or wait for operator evidence.

## 2. Start the root task

1. In ChatGPT web/desktop, select **Work in → Cloud**, choose the published WP0-verified environment and start a new task. This root task is the run's execution context. The factory does not create it programmatically.
2. Pass only run ID, approved specification reference/SHA, repo refs, privacy class and operator-authorized objective. Do not paste secrets or unnecessary product source.
3. PO validates input, pins immutable requirements, initializes the run manifest and records an explainable org/resource decision before child work.
4. PO selects the smallest organization justified by complexity/dependencies/risk. PO plus Worker is valid for simple work; optional managers require explicit reason.
5. Before every spawn, create and validate the assignment, reserve parent resources and check WP0 concurrency. Ask Codex to spawn the child natively in the same root task. The parent waits and reviews result.

## 3. Execute and accept

For each assignment:

1. Parent supplies exact task objective, requirement IDs, context refs, allowed paths, dependencies, deliverable, checks, authority, resource reservation, stop/retry/escalation conditions and pinned instruction refs.
2. Worker implements only assigned product scope, runs specified checks, commits/records product output according to repo policy and writes a factual factory journal. A Worker cannot change factory policy or accept its own result.
3. Parent records result and evidence. It accepts or issues a bounded change request with a new attempt/event; repeated failure fingerprint triggers manager review.
4. Parent escalates cross-scope conflicts, resource issues, external criteria changes, permission errors and protected actions to the correct authority.
5. PO tracks requirement closure and integration dependencies across branches. Criteria changes require customer approval and must be versioned; never overwrite source criteria to make a test pass.
6. PO checks integrated product against external acceptance criteria and system-level evidence. Record separate product acceptance from local task acceptance and Cloud session state.

## 4. Monitor resources and pause

At each planning/delegation boundary, inspect shared ledger and current usage screen when available. Record measured, estimated, reserved and unknown separately. Reservations are local policy, not provider enforcement.

If allowance/headroom reaches the authorized stop threshold, or the remaining usage cannot be verified under policy:

1. Stop new subagent fan-out.
2. Finish only safe/in-flight idempotent work; do not start an irreversible action.
3. Checkpoint both repo refs, active/accepted assignments, child status/thread refs, resource events, open saga intents, pinned versions and explicit next action.
4. Set run/work state to `PAUSED`; leave unfinished work recoverable.
5. Tell the human/operator what evidence is needed for resume. Do not purchase credits, enable auto-reload, switch to API billing or auto-reactivate at reset.

## 5. Finish and close

1. Resolve every required work item to accepted, explicitly blocked, cancelled or externally deferred with authorized decision.
2. Verify all required acceptance criteria and integration evidence; customer approval is captured where needed.
3. Reconcile both repository heads, PR/commit refs and open saga operations. The run cannot be accepted with an ambiguous side effect.
4. Ensure public factory records are privacy-reviewed; run secret-canary checks and human review where content or metadata is uncertain.
5. Produce a report of P/A coverage, final state, repo refs, accepted artifacts, outstanding risks, decisions, resource classes and next action (none for a closed run).
6. PO records `ACCEPTED` only after integrated criteria pass. The operator/human reviews release/publication and protected operations.
7. Archive the completed run in Git while preserving all prior events and active-to-archive refs.

## Records to preserve

Keep the current manifest, all immutable assignment versions, decisions/approvals, execution journals, resource ledger, sync operations, acceptance records, experiment refs, source spec SHA, both repository refs, policy/model/skill hashes, Cloud task/thread IDs when visible and the last recovery action. If a value is not exposed, record `unknown` with reason instead of omitting it or guessing.

Related contracts: [data contracts](../interfaces/data-contracts.md), [state and recovery](../architecture/state-and-recovery.md), [CLI](../interfaces/command-line-interface.md), [resource management](../architecture/resource-management-and-self-correction.md).

