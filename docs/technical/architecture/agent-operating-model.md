# Agent operating model

## Role semantics

Roles are bounded authorities, not long-running services. A run always has a root PO function and at least one product Worker. Chief and Planner are optional and selected when coordination benefit justifies their context and resource cost. The supported org shapes are:

```text
Simple:       PO ── Worker(s)
Scoped:       PO ── Planner ── Worker(s)
Multi-track:  PO ── Chief ── Planner(s) ── Worker(s)
Direct tracks (where supported): PO ── Planner(s) ── Worker(s)
```

These are effective Codex subagent trees. The PO is the root task agent; every delegated child is created through native Codex subagent capability within that same root task. The selected runtime must pass WP0 for nesting, simultaneous threads and model behavior before claiming support. No child assignment moves to a separate Cloud task.

| Role | Scope and duties | May delegate to | Acceptance / escalation | Prohibited |
|---|---|---|---|---|
| Human customer/operator | Approve external requirements, launch/resume root task, decide protected actions and PO-core/governance changes | Root PO by authorizing work | Final human approvals and unresolved architecture decisions | Silent action by agent on human's behalf |
| PO (root) | Whole product, source-of-truth requirements, organization choice, shared contracts, integration, requirement coverage | Chief, Planner, Worker, as justified and Cloud-verified | Accepts integrated product; escalates external changes/protected actions to human | Change PO core prompt or immutable policy without human approval; silently change acceptance criteria |
| Chief (optional) | Cross-track architecture coordination, shared budget and Planner oversight | Planner(s), Worker(s) where useful | Accepts direct reports; escalates product-wide or out-of-authority choices to PO | Override PO contracts; change own base prompt |
| Planner (optional) | Explicit direction/epic; decompose; reserve resources; direct Worker supervision | Worker(s) | Accepts direct Worker assignments; escalates cross-track decisions to Chief/PO | Change scope/contract outside assignment; approve own base prompt |
| Worker (required) | One executable assignment, scoped artifact, relevant checks and factual journal | None in v1.0 unless explicitly granted the leadership role in a separate approved assignment | Reports blocked/failed work to parent; parent accepts local result | Self-accept, factory meta-analysis, self-edit base prompt, propose factory-wide policy from worker journal |

An agent that plays a role still acts under an immutable assignment and authority snapshot. The role name alone does not grant write access.

## Authority and instruction precedence

Instruction precedence for a run is:

1. Platform permissions, immutable safety/privacy/budget constraints and human approvals.
2. Role authority and the approved authority matrix.
3. Allowlisted product-local instructions pinned to a product SHA, when applicable.
4. General factory role instructions, policies and skills.
5. Run/task-specific instructions issued by an authorized parent within the above scope.

P-010 gives product-specific instructions precedence over conflicting general factory instructions within the authorized task scope. This does not let product content override the first two layers, grant new repo permissions or modify external acceptance criteria. Conflicts are recorded with sources, affected requirement, proposed resolution and decision authority. The PO resolves product-wide cross-component contracts; isolated Planners do not renegotiate them privately.

## Assignment contract

Before spawning a child, the parent records and validates an assignment containing:

- stable `run_id`, `assignment_id`, `task_id`, parent ID and target role/agent reference;
- immutable requirement IDs and objective;
- allowed context references and path scope, including product repo ref/SHA;
- dependencies and blockers;
- expected artifact paths/commit or PR evidence;
- explicit acceptance checks and evidence format;
- authority snapshot and pinned factory/product instruction hashes;
- requested model/reasoning and actual observed model/reasoning, or `unknown`;
- resource estimate/reservation, retry cap, stop conditions and escalation conditions;
- native thread/agent identifiers when visible; absent platform evidence is `unknown`.

The child receives only the context needed for the assignment, restates its boundary when material, performs only assigned work and returns artifact refs, checks, deviations, failures, usage classification and blockers. It must not invent missing acceptance criteria. Parent-child requirement closure is checked before delegation and after results; no child can drop parent criteria or create silent scope expansion.

## Native delegation protocol

1. Parent checks the current manifest/repository refs, parent budget and Cloud concurrency evidence.
2. Parent writes the versioned assignment and reserves shared resources before asking native Codex to spawn the child.
3. Parent asks Codex to spawn the configured role (or uses a generic native spawn only if WP0 proves equivalent behavior) with the exact assignment ID and role instruction reference. The written assignment is the durable contract; the native prompt is its execution envelope.
4. Parent waits for the child thread, captures its thread/result reference when available, validates output and records the outcome.
5. Parent performs local acceptance or a bounded change request. Retries use a new attempt/event and preserve prior output. A repeated fingerprint or exhausted retry cap triggers replan/escalation/pause.
6. Parent stays responsible for work assigned below it; completion of a child thread is not acceptance.

The process must remain inside one root Cloud task even for recursive Planner/Chief delegation. If actual Cloud cannot provide required nesting, model behavior or concurrency for A-03, stop for architecture review. See [Codex Cloud verification boundary](../platform/codex-cloud.md).

## Acceptance and escalation

Local acceptance belongs to the direct parent and uses the assignment's pinned criteria. Record accept/reject/change-request, reviewer identity/role, evidence references, requirement coverage and timestamp. PO integration acceptance is separate and checks the external product specification, integration contracts, unresolved requirements and system-level evidence. Passing unit tests alone cannot satisfy external product acceptance.

Escalation carries: problem and impact, affected requirements/tasks, evidence/source refs, actions already taken, remaining options, resource/retry state, risk, decision requested, deadline/boundary if any, and the receiving authority. A role resolves issues within its delegated authority; it does not pass a decision down to a less-authorized child.

## Assignment immutability and reorganization

An assignment version is immutable after it has been used. Corrections create a new version and reference the superseded assignment. Reorganization may add/retire managers or reparent remaining work, but completed task IDs, accepted artifacts, prior decisions and source events remain unchanged. New assignments reference the reason, new authority and preserved requirement mapping (P-016).

## Configuration references

The native configuration contract is:

- Root `AGENTS.md` selects the PO role procedure, points to immutable factory policies and requires native delegation in the same task. PO is the root role; it is not a child custom-agent file.
- `.codex/config.toml` enables agents. When project config is honored in Cloud, `[agents]` sets `enabled = true`, `interrupt_message = true` and `max_concurrent_threads_per_session` to the integer evidenced in WP0, capped at six and never above the actual accepted runtime limit. Leave default child model/reasoning settings unset so children inherit root settings unless WP0 verifies and justifies a supported override.
- Project-scoped `.codex/agents/chief.toml`, `planner.toml` and `worker.toml` each define unique `name`, role-specific `description` and `developer_instructions` that point to the canonical role file and relevant skills. Optional `model` and `model_reasoning_effort` are allowed only when that specific combination is evidenced in Codex Cloud. No child file claims to override root provider, MCP, approval or sandbox/permission settings.
- Role procedures are versioned at `factory/roles/po.md`, `chief.md`, `planner.md` and `worker.md`. Repo skills live at `.agents/skills/start-run/SKILL.md`, `delegate/SKILL.md`, `review-result/SKILL.md`, `integrate/SKILL.md` and `recover-run/SKILL.md`; each skill declares `name` and `description`.
- Feature reductions in custom agent files are permitted only for a capability shown unnecessary and tested as honored in Cloud. Prompt/path boundaries are not a substitute for sandbox/permission enforcement.

Official Codex configuration documentation describes these settings/files for Codex clients, and repository skills are documented as available to Cloud tasks. It does not establish that project TOML or custom-agent model settings are loaded in every Codex Cloud task. Therefore WP0 must test discovery, effective settings, nesting and inheritance. If project configuration is ignored, use a generic native role prompt in the same root task only if it preserves required role and model behavior; otherwise the architecture is blocked. See [Codex Cloud evidence](../platform/codex-cloud.md).

