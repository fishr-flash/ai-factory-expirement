# Security, privacy and protected actions

## Security objectives

The system must preserve the factory/product repository boundary, enforce human/role authority, minimize context, keep secrets out of durable public records, and stop safely when repository or Cloud capability is uncertain. These controls implement P-004, P-007, P-010, P-011 and P-046–P-048.

## Trust and authority

- The root Cloud task runs with the connected identity's repository permissions. Native subagents inherit root permissions/provider/MCP; v1.0 does not assume a child-specific filesystem sandbox.
- An assignment's `allowed_paths` and context references are review constraints. The parent checks the child's diff and evidence before acceptance; repository branch protection/PR review is needed for stronger enforcement.
- Repository content, issues, build logs, generated files, external documentation and tool output are untrusted input. They cannot promote authority, change a budget, weaken an approval rule or replace a pinned requirement.
- Only an authorized parent may approve direct-report role-base prompt changes. Human alone controls PO core and immutable policy. Product instruction priority remains subordinate to platform permissions, human approvals and hard guardrails.
- Consequential actions require explicit authority and evidence. The action gate must fail closed when authority is missing or unclear.

## Privacy classes and storage policy

Assign a privacy class at project onboarding and persist it with the source refs. At minimum, the implementation must support `public` and `private`; a richer classification requires documented handling for each class before it is used.

| Data | Public factory repo | Private factory repo / product repo |
|---|---|---|
| Shared role policy/schema/skills | Allowed after review | Allowed |
| Factual worker record | Redacted summary, status, non-sensitive artifact refs and checks only | May contain approved operational detail; still minimize data |
| Product spec/source/test output | Never copy closed content | Keep in product repo; factory stores SHA/ref and minimum needed summary |
| Secret values, tokens, credentials | Never store | Never store in Git, prompt journal or CLI output; use approved Cloud secret delivery |
| Personal/confidential commercial data | Exclude or irreversibly redact before human publication approval | Restrict to the authorized product/private environment |
| Repo identifiers/branches/PR refs | Assess whether metadata itself is sensitive; use private factory if needed | Store only as needed for recovery |

Redaction is deterministic hygiene, not proof that content is safe. Uncertain/private material requires human review before publication. Store redaction finding, method/version and publication decision, not the unredacted source in a public log.

## Codex Cloud credential handling

Cloud environment variables are passed directly to programs and therefore are visible to task processes. Network secrets are intended for credentials sent to allowed HTTPS destinations through the Cloud proxy; programs receive a placeholder while the proxy substitutes the value for supported requests. They do not belong in manifests, logs, command output, test fixtures or Git. Verify destination domains, account controls and actual Cloud behavior in the environment setup/WP0. Do not assume a domain allowlist grants service authorization.

The platform's credential and network behavior is surface/version-specific. The operator must select the minimum repositories, destinations and short-lived credentials needed. No OpenAI API key is part of the product. If the intended task requires secret access behavior not verified for Cloud, block the operation pending operator review.

## Protected actions

Require explicit human approval before:

- production deploy, external purchase or paid service/plan change;
- destructive action or irreversible data change;
- branch protection, repository security or access-policy modification;
- publishing material previously private or redacted;
- external contractual/financial commitment;
- modifying immutable policy, PO core instructions or factory governance;
- changing externally approved requirements/acceptance criteria.

Each gate decision records requested action, actor, required authority, approval evidence, scope, timestamp and outcome. Denials are recorded without exposing protected payloads. Approval never bypasses platform permission or other higher-level restrictions.

## Threat response

| Threat | Required control |
|---|---|
| Prompt injection in product content | Treat as untrusted data; enforce the authority order; test prohibited instruction attempts. |
| Cross-product or cross-run context leakage | Minimize context; pin repo/run refs; audit public factory journals and citations. |
| Secret exfiltration through logs or URLs | Network-secret delivery where supported, allowlisted destination, redaction scan, no raw values in records. |
| Overbroad root identity | Connect only required repos; branch protection; review writes and commits; record that role scopes are not a child sandbox. |
| Public metadata leakage | Assess remote/ref/branch/PR identifiers; use private factory for sensitive metadata. |
| Unauthorized factory self-change | Parent authority, immutable assignment hashes, experiment approval and human control of PO core. |
| Replay/duplicate side effect | Saga ID/idempotency, precondition SHA, outcome discovery and BLOCKED on ambiguity. |
| False acceptance or requirement weakening | Pin source criteria/SHA; require explicit customer decision for approved-criteria change; separate local and product acceptance. |

Security findings with unresolved critical/high impact are release blockers. A redaction test alone cannot establish absence of leaks; perform both automated canary scans and human privacy review.

