# WP0 Codex Cloud capability evidence

**Status:** NOT RUN in this technical-documentation task. This file defines the evidence record for the still-open WP0 gate; an empty result is not a pass. Do not fill it with inferred behavior or private account identifiers.

## Purpose

WP0 determines whether the intended ChatGPT account and Codex Cloud surface support the already-approved AI Factory v1.0 architecture. The account/runtime test supplements official docs; it does not change P/A scope. If required topology, repository writes or model/role behavior fails, stop implementation and request a human architecture decision. No API path or manual child-task relay is allowed.

Run the procedure in [Codex Cloud platform boundary](../platform/codex-cloud.md#wp0-capability-gate) and [environment setup](environment-setup.md). Use a disposable product repository and synthetic data only.

## Evidence header

Record for each WP0 campaign:

| Field | Value to record |
|---|---|
| Evidence ID | Unique `wp0_…` ID |
| Start/end time | ISO 8601 UTC |
| Account/plan class | Non-identifying class and workspace policy; no email/name/account number |
| Cloud surface/client | ChatGPT web/desktop and visible product mode/version if available |
| Environment | Non-sensitive environment name/ref, publish date and setup hash if available |
| Repositories | Logical repo IDs and start refs; redact remotes if public metadata is sensitive |
| Operator | Role/initials or approved non-identifying actor ID |
| Evidence refs | Sanitized screenshots, UI/task/thread references and commits; never include credentials/source |
| Official docs checked | URLs and access date |
| Conclusion | `PASS`, `BLOCKED`, `UNKNOWN`, or `NOT RUN`; include decision owner if blocked |

## Capability observations

| Capability | Procedure | Result to capture | Status |
|---|---|---|---|
| ChatGPT-plan Cloud eligibility | Start/select published Cloud environment from intended account. | Eligibility, any plan/workspace restriction and exact surface. | NOT RUN |
| Two repository checkout | Select factory plus disposable product repo in one environment/task. | Both checkout refs, branch names, read/write outcome, connected identity, commits/PR refs and independent histories. | NOT RUN |
| Native child spawn | Spawn one child inside a single root task. | Root/child IDs if visible, role instructions received, completion/result behavior. | NOT RUN |
| Nested depth | Ask child to spawn grandchild; continue required PO→Chief→Planner→Worker path. | Deepest accepted tree, failure boundary/error, whether nested child can itself delegate. | NOT RUN |
| Required topology | Exercise PO→Chief→2 Planners→1 Worker per Planner. | Each role/thread ref, assignments, parent-child hierarchy and accepted work in one root task. | NOT RUN |
| Simultaneous child threads | Increase open child threads, attempt through six, then one beyond if permitted. | Maximum open threads excluding root; when a thread counts as open; accepted/rejected behavior. Six is only initial target. | NOT RUN |
| Model inheritance | Spawn child and nested child without explicit model/effort. | Root selection, requested/observed child model and effort, inheritance evidence; use unknown when not observable. | NOT RUN |
| Model override | Request a different available model/effort on child and nested child if UI/runtime permits. | Available model list for Cloud surface, accepted/rejected override, actual observed model/effort and evidence source. | NOT RUN |
| Project config/custom agents | Verify root `AGENTS.md`, `.codex/config.toml`, `.codex/agents/*.toml` load/effect. | Observed effective fields, precedence, config warnings and runtime cap. Do not infer from local parser success. | NOT RUN |
| Repository skills | Invoke known repo skill under `.agents/skills/<name>/SKILL.md`. | Discovery/invocation evidence, path and any scope restriction. | NOT RUN |
| Permission inheritance | Attempt allowed and denied behavior using disposable files. | Root/child permissions, repo access and whether child restrictions are policy-only. Do not use production secrets. | NOT RUN |
| Network and secrets | Test approved disposable value at a permitted HTTPS domain and a denied domain. | Placeholder/proxy behavior, allowed-domain enforcement, visible environment variables, logs and denial. | NOT RUN |
| Plan usage | Observe account usage before/after controlled task(s), check reset display. | Usage evidence granularity, reset view and whether agent/run attribution exists. Unknown when UI cannot prove it. | NOT RUN |
| Same-task continuation | Pause/close then reopen the same task where safe. | Task state and file/VM persistence behavior; any loss. | NOT RUN |
| Saved VM retention | Record documented/current retention statement and any observed expiration behavior. | Actual account/current docs and evidence; do not depend on VM retention for archival. | NOT RUN |

## Result record

For each row, append a result with:

```text
Capability ID:
Attempt timestamp:
Expected behavior:
Observed behavior:
Evidence reference:
Status: runtime-verified | blocked | unknown
Affected P/A IDs:
Conclusion and limit:
Follow-up/owner:
```

These lines are a documentation template only, not a machine-consumable runtime record. When implementation exists, store the same sanitized facts in its versioned run evidence contract and link the commit here.

## Gate decision

### Required pass conditions

- The account can run the selected ChatGPT-plan Cloud task.
- Both repositories are available and separate writes/history can be reconciled.
- The required PO→Chief→2 Planners→Workers topology and necessary concurrency work as native subagents inside one root task.
- The intended roles, instructions and required model behavior can be represented without unsupported assumptions; unavailable/unobservable model settings are recorded as unknown only if the accepted design permits them.
- Parent permission inheritance and secret/network behavior do not violate the privacy, least-privilege and approval rules.
- An operator can observe enough plan availability to apply the approved stop/pause policy; when detailed attribution is unavailable, ledger values remain unknown and no hard cap is claimed.

### Decision

**Current decision: OPEN / NOT RUN.** Record `PASS` only after each required condition has evidence. For `BLOCKED`, enumerate failed capabilities, affected P/A IDs, options that preserve full scope, consequences, and the decision required from the human. Do not record a pass because a local Codex run or published config sample succeeded.

