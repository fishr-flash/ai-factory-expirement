# Resource management and controlled self-correction

## Resource model

Resources form a parent-child budget tree. PO owns the run allocation unless a Chief is active; Chief allocates to Planners; each Planner allocates/reserves for its child assignments. Estimates, reservations, observed usage and unknown values are separate fields. Use explicit resource type, unit and measurement window; never compare unlike units or assume plan usage is linear over time.

The subscription provider enforces account-level plan limits. AI Factory has no documented mechanism for hard caps per run, agent or role. The ledger is a conservative, auditable policy gate, not an enforcement proxy. It can limit planned fan-out and require an operator observation before work, but it cannot guarantee that a sibling or another conversation will not consume shared plan usage.

## Reservation and fan-out decision

Before every native child spawn, the parent performs the following decision:

1. Read current open spawned-agent threads and the maximum verified by WP0 for the intended Cloud surface.
2. Read the parent/run allocations, outstanding reservations, latest usage evidence and retry limits.
3. Choose a reservation unit that is supportable (for example, task/thread count or dashboard-observed usage); record its source, reliability and validity window.
4. Reserve from the parent before spawning. Reject if the request exceeds its parent allocation, the WP0 concurrency cap, or the configured organization fan-out limit.
5. If current shared headroom cannot be verified under the selected policy, do not add parallel siblings. Complete a safe checkpoint and move the run to `PAUSED` or request human evidence; do not treat an estimate as observed usage.
6. On completion/interruption, append release/observation/correction events linked to the reservation. Never edit prior ledger rows.

The policy threshold is configured by an authorized owner based on actual UI/account evidence. The design does not specify a universal percentage or fixed credit-to-agent conversion. No additional credits, auto-reload, API billing or plan change is initiated by the system.

## Usage classification

| Class | Meaning | Example/source |
|---|---|---|
| `estimated` | Forecast, not measured | Parent estimates a scoped assignment from comparable work. |
| `reserved` | Soft allowance assigned in the factory ledger | Child reservation and hierarchy path. |
| `observed` | Value directly visible from an accepted provider/dashboard source | Include source and observation timestamp/window. |
| `unknown` | Not exposed or attribution not reliable | Store `null`, status unknown and why/source unavailable; never substitute 0. |

Reconcile estimated vs observed values when comparable. Metrics must include sample size and comparability. If multiple tasks share one plan-level figure and attribution cannot be separated, it stays unallocated/unknown rather than apportioned as fact.

## Retry and stop policy

Each assignment has a finite retry cap, current attempt index, retry categories and stop/escalation conditions. Every retry records the previous failure fingerprint, evidence, changes attempted and expected differentiating evidence. Two consecutive occurrences of the same failure fingerprint trigger manager review. At retry exhaustion, the parent chooses a bounded split/replan, escalation, pause, cancellation or explicitly authorized new assignment version; there is no open-ended automatic retry loop.

Transient idempotent operations may use bounded backoff. Authentication/permission errors, quota exhaustion, invalid requests, changed SHA and ambiguous side effects are not blindly retried. See [incident and recovery](../operations/incident-recovery.md).

## Pause and resume budget gate

When the account limit/window is exhausted or the safety threshold is reached:

- stop creating new child assignments;
- finish or interrupt work only at a safe boundary;
- checkpoint run state, both repo refs, child statuses, reservations, open operations and next action;
- mark run/work as `PAUSED`, not `FAILED`;
- record actual plan evidence if visible, otherwise unknown;
- require a human-authorized continuation and fresh evidence/reconciliation before returning to `ACTIVE`.

Reset time does not trigger automatic task launch or reactivation. The operator checks the same account's current limits and resumes through the ChatGPT UI workflow. See [production runbook](../operations/production-runbook.md).

## Factory improvement lifecycle

Leaders, not Workers, interpret execution records and identify repeatable inefficiency. Findings are classified as product-only or candidate factory-wide (P-012). The improvement state machine is:

```text
OBSERVATION → PROPOSAL → AUTHORIZED_EXPERIMENT → EVALUATION
                                        ├── ADOPTED
                                        ├── REJECTED
                                        └── ROLLED_BACK
```

Before changing any role-base prompt, skill or shared method:

1. Identify source events, affected requirement/task classes, scope (product or factory), and the authorized parent.
2. Record baseline version/hash, candidate diff/hash, hypothesis, expected effect, success metric, guardrails, rollout cohort, sample limits and rollback trigger.
3. The direct parent approves direct-report prompt changes. Human approval is required for PO core, immutable policy and protected governance.
4. Keep in-flight assignments pinned. Only future assignments/runs use the new approved version unless a documented restart decision is made.
5. Evaluate on comparable task classes/complexity and record model/tool versions, acceptance, rework, retries, escalation, recovery and measured/estimated/unknown usage.
6. A single incident cannot prove general factory benefit. Cross-product adoption needs replicated or otherwise comparable evidence and correct authority.
7. Adopt, reject or rollback with rationale. Rollback is a new Git version with lineage, not history erasure.

## Metrics and report semantics

The v1.0 report must allow leaders to assess these product/factory quality indicators:

| Indicator | Minimum representation |
|---|---|
| Product acceptance and integration correctness | External criteria pass/fail/unknown, integration evidence refs, PO decision and gaps. |
| Implementation/review cycles | Count of attempts/change requests per accepted task and failure/rework categories. |
| Resource planning accuracy | Estimates, soft reservations, observations and unknowns by resource/unit/window; compare only comparable periods/units. |
| Cost per accepted result | Report only when source data can be reliably attributed to an accepted result; otherwise provide a declared nonfinancial measure or `unknown`. |
| Org/model effectiveness | Outcomes grouped by comparable task class/complexity, role tree, model/effort and tool version; model actual can be unknown. |
| Escalation, pause and recovery | Counts, reason, resolution, time-to-resume where observable, duplicate/repeated work and outcome. |
| Instruction/skill evolution | Change provenance, hypothesis, baseline/cohort, quality/resource effect, regressions and adoption/rollback state. |

Reports distinguish product acceptance from provider/session completion and state each denominator, cohort and missing value. A costlier or more complex later project alone is not evidence of factory regression. Do not infer improvement from fewer tokens alone or compare unlike task classes without stratification.

