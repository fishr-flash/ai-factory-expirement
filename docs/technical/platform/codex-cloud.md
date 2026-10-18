# Codex Cloud platform boundary

## Purpose and evidence policy

The v1.0 provider/runtime choice is Codex Cloud under an eligible ChatGPT plan, used through the ChatGPT UI. The official documentation changes over time and availability varies by account, plan, region, client surface and workspace policy. This document labels claims by surface and separates published documentation from evidence that must come from the intended Cloud account.

**Evidence reviewed:** official OpenAI Learn pages below, reviewed 18 October 2026. The user's intended account has not been runtime-verified in this documentation pass. An implementer must recheck the linked pages and complete WP0 before treating an account-specific behavior as available.

## Documented behavior

| Capability | Current documented statement | What it does not prove |
|---|---|---|
| Hosted task | Codex Cloud runs tasks in a hosted environment; work can continue while the user's computer sleeps. A published environment contains selected repos, dependencies, tools and access settings; new tasks receive isolated workspaces. | The intended account has access; every product's repo combination works; a session remains available indefinitely. |
| Repository selection | Environment setup selects GitHub repositories. Cloud task workflow can review, commit or open a PR. | A factory+product two-repo change and durable writes to both in one task; exact connected identity/scope. WP0 must test both. |
| Saved VM | Official environment docs state a task's saved VM is recoverable by default for up to seven days after the last turn/resume. | A universal SLA or recovery after that window. Git remains the long-term system of record. |
| Repository skills | Skills stored in a repository are available in Cloud tasks; personal local skills are not synced. | That a particular skill is loaded, invoked, or honored by the target task; WP0 checks paths and behavior. |
| Network/credentials | Environment controls allowed domains. Direct environment variables are passed to programs. Network secrets deliver a placeholder to the task program and proxy-substitute credentials for supported HTTPS requests to allowed domains. | That a domain has authorized the task; that secret access/port/path behaves correctly for this use; that raw env vars are hidden from task programs. |
| Plan usage | Cloud chats share plan usage with local messages; usage dashboards show current limits/reset information. Subagent work can use more tokens than a comparable single-agent task. | Reliable per-run/per-role consumption, universal rate or budget conversion, or a factory hard cap. |
| Subagents | Official Codex/ChatGPT Work documentation describes spawning specialized agents in parallel and collecting results. Custom Codex agent files/config and per-agent model/reasoning settings are documented for local Codex clients. | That `.codex/agents`, project `config.toml`, child-specific model overrides or arbitrary nesting are loaded/effective in the intended Codex Cloud root task. This is a WP0 gate. |
| Models | Model availability is surface- and identity-specific. Official workspace guidance says Codex Cloud model access follows models supported by hosted workflows and the signed-in identity; it also states Cloud does not support changing its default model. | A custom model per subagent or per-role in a Cloud task. Test actual task controls and observable settings; mark unavailable/unobservable as such. |
| Browser/computer use | Current Cloud environment documentation lists computer and browser use among unsupported Cloud environment capabilities. | UI/browser testing in the Cloud task. Use local automated tests or explicitly assigned human review unless docs and target account change. |
| Programmatic task launch | Current Developer Commands documentation lists `codex cloud exec` and `codex cloud list`. | Permission under the selected user plan, a stable product integration contract or approval to automate launch. AI Factory v1.0 intentionally excludes these commands and uses human UI launch under D-03. |

## Approved product boundary

The product must use included ChatGPT subscription usage and may not use OpenAI API billing, API keys, Agents/Responses API, an API app-server, external orchestration service or automatic credit purchase. Root task creation/resumption is human-operated through the selected ChatGPT UI path. This remains the approved design even though current platform docs expose some programmatic Cloud CLI commands; adding automation would require an explicit architecture/scope decision.

Native subagents within a single root Cloud task are the required delegation mechanism. Separate tasks are used only when a human authorizes recovery after loss of saved VM; that recovery uses the same run ID and Git state and cannot relay child assignments. If nested child spawning in Cloud is unavailable, that is an architecture blocker, not an invitation to use manual task relay.

## WP0 capability gate

Record each attempt using the [WP0 evidence record](../operations/wp0-evidence.md). Use a disposable product repo and sanitized evidence. At minimum, verify:

1. Actual account access to Cloud under the intended ChatGPT plan and workspace policy.
2. Published environment selects both the factory and disposable product repositories and the task can read/write each under the connected identity. Record commits/PR refs and recovery behavior separately.
3. One root task can spawn a native child which can spawn a grandchild; exercise `PO → Chief → 2 Planners → 1 Worker per Planner` and record deepest successful tree and maximum simultaneous open child threads. The initial example value six is only a configuration target, not a Cloud guarantee.
4. Root/child/nested model and reasoning selection/inheritance: record requested value, actual observable value and evidence surface. Do not infer a child override from local TOML documentation.
5. Whether root `AGENTS.md`, `.codex/config.toml`, project `.codex/agents/*.toml` and repository `.agents/skills/*/SKILL.md` load and affect role behavior. When custom profile loading fails, test a generic native spawn in the same task only if it still supports all required model/role behavior.
6. Parent/child permission inheritance, connected repo permissions, network controls and secret delivery; use non-sensitive test values and only approved domains.
7. Plan allowance/reset visibility and attribution; record per-run/per-child usage as unknown if the interface cannot prove it.
8. Same-task reopen, task interruption and saved VM behavior. Record actual account behavior; do not depend on it for long-term recovery.

WP0 passes only when required topology, repository access and necessary model/role behavior are supported in the selected runtime. An inability to verify a claim is not a pass. If required support fails, stop and return evidence plus the decision required to the human. Never silently reduce A-03/P-015 or substitute API billing or another Cloud task.

## Cloud environment operating requirements

- Publish a reusable environment only after setup and repository access review; a published environment is a starting image, while an existing task retains its own task state.
- Keep the environment's selected repo list, dependency setup, domain allowlist, direct environment variables, network secrets and sharing scope minimal. Re-publish after setup changes and verify a new task.
- The repository connection determines GitHub access. An environment's domain allowlist does not grant service permissions.
- Prefer network secrets for credentials sent to supported services; remember any direct environment variable is visible to task programs. Never put values in Git or records.
- Keep personal/local skills out of dependency assumptions; required skills must live in versioned repo files.
- Do not treat the provider VM, transcript, activity view, or local `~/.codex` configuration as durable factory state.

## Sources

- [Codex Cloud overview](https://learn.chatgpt.com/docs/cloud)
- [Cloud environments](https://learn.chatgpt.com/docs/environments/cloud-environments)
- [Codex subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- [Pricing and usage](https://learn.chatgpt.com/docs/pricing)
- [Workspace model availability](https://learn.chatgpt.com/docs/enterprise/workspace-model-availability)
- [Developer commands](https://learn.chatgpt.com/docs/developer-commands)

