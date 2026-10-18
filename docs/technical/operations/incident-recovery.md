# Incident response and recovery

## First response

When a run stops unexpectedly, do not immediately retry the last action. Preserve the state and identify whether the failure happened before or after a durable write.

1. Stop new fan-out and any non-idempotent action.
2. Capture non-sensitive Cloud UI/task/thread status, last known turn, error and visible usage/limit information.
3. Read the last committed factory checkpoint and compare factory/product refs. Do not infer from uncommitted workspace state.
4. Validate schema, events, current state and open sync operations with `ai-factory validate` and read-only `status`/`reconcile`.
5. Assign the failure category below. Record event IDs, evidence source, actor and current refs. Redact secrets/private product content.
6. Escalate to the authority that can resolve it. Keep run in `PAUSED` or `BLOCKED` until a safe path is authorized.

## Incident matrix

| Category | Immediate response | Resume condition |
|---|---|---|
| Usage exhausted/window closed | Stop spawns; checkpoint and mark `PAUSED`; operator checks plan usage/reset. | Fresh usage/headroom evidence or human-approved unknown policy, plus refs reconciled and explicit continuation. |
| Cloud task interrupted but saved state available | Reopen the same task through ChatGPT UI; compare task state with Git checkpoint. | Reconciliation finds no state loss or contradictory side effect. |
| VM expired or unavailable | Human starts a recovery task for the same run ID in published environment; use Git only. | State, refs and next action reconstruct; no child assignment is being relayed. |
| GitHub authentication/permission failure | Stop writes and secret use; record safe error details; operator reviews repo connection/access. | Correct identity/scope verified in a disposable check; no API fallback. |
| Factory SHA conflict | Do not overwrite; inspect competing commits and operation IDs. | Parent-authorized merge/replan creates new expected SHA and audit event. |
| Product SHA conflict | Do not rebase/replay blindly; inspect product diff and PR status. | Product owner/parent authorizes conflict resolution and saga records new refs. |
| Partial saga / product commit without factory result | Search commit/PR and idempotency key; append missing factory result if proven. | Both repository refs and intent agree or ambiguity is resolved by PO/human. |
| Ambiguous external action | Set operation/task `BLOCKED`; do not replay. | Human/PO confirms actual side effect and authorizes resolution. |
| Schema or manifest/journal mismatch | Preserve files, prevent state advancement and auto-retry. | Versioned correction/superseding record is reviewed and validation passes. |
| Suspected secret/privacy leak | Stop publication and further copying; contain repository/credential exposure under the owner's process; keep details out of public log. | Credential owner confirms rotation/containment, privacy review closes findings and action is authorized. |
| Prompt injection/authority violation | Treat source as untrusted; stop protected write, capture source ref and violated rule. | Parent/human reviews and assignments/policies are validated; no authority change from repo content. |
| Repeated same failure fingerprint | Stop automatic retry; escalate to parent with attempts/evidence/changes. | Parent records split/replan, changed assignment, pause, cancel or approved new attempt. |

## Git-only recovery procedure

After saved Cloud VM state is unavailable:

1. Operator starts an authorized recovery task through the approved ChatGPT UI with the published environment and same factory/product repositories. This is not a child task and must not receive a pending child assignment as ordinary delegation.
2. Load committed `manifest.json`, event journals, assignment versions, resource ledger, decisions, acceptance records and open sync operation records for the run.
3. Read current factory and product refs. Validate commit identity and scan for idempotency keys/operation fingerprints.
4. Compare source requirements, allowlisted product instructions, factory role/policy/skill hashes and model catalog refs against the pinned versions. Any changed input requires explicit decision before resuming.
5. Reconstruct state, accepted work, retries, active work, plan usage status, child thread refs (unknown if unavailable), open operations and `next_action`.
6. Apply the reconciliation algorithm in [state and recovery](../architecture/state-and-recovery.md). Mark uncertainty as unknown/blocked; do not recreate work whose completion can be proven.
7. Capture a new recovery checkpoint and operator authorization. Resume the same `run_id` only after state is consistent and allowed headroom is confirmed or an authorized unknown policy applies.

## Incident record

Every incident record includes incident/run/task/operation IDs; detection time; current repo refs; affected P/A requirements; actor/authority; category/severity; evidence references; actions and retries; resource values by class; whether privacy/protected action is involved; decision/owner; safe next action; resolution and prevention experiment if authorized. Keep raw secrets, credentials, confidential source and unnecessary user data out of shared records.

## Close criteria

Close only after the run state is reconciled, no ambiguous external side effect remains, required approvals are captured, sensitive material is contained, next action is explicit (or terminal), and the owning authority approves closure. The production run can remain paused/blocked after incident response if recovery conditions are not met.

