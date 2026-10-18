# Environment and repository setup

## Scope

This runbook prepares a reusable Codex Cloud environment and the two repositories required by AI Factory. It does not create a Cloud task or make a product change. The operator performs account actions in ChatGPT and GitHub. See [Codex Cloud evidence limits](../platform/codex-cloud.md) and the [CLI boundary](../interfaces/command-line-interface.md).

## Prerequisites

- A ChatGPT account/workspace for which Codex Cloud access is enabled and usage/limits can be reviewed.
- GitHub connection with least-privilege access to the factory repository and the specific target product repository.
- Factory repository on a supported GitHub host; public/private status and privacy class known.
- Product repository with externally approved specification, acceptance criteria, constraints and product-local instruction allowlist.
- Branch protection and PR/review policy set according to the consequence of changes.
- Environment secret/domain/network requirements reviewed; no credential values in repo files.
- WP0 evidence supports the required subagent topology and the exact intended repo setup. If not, stop and request a decision.

## Prepare repositories

### Factory repository

1. Use the reusable AI Factory repository. Do not place product application source here.
2. Choose a public factory only if all run identifiers, journals, refs and evidence can be made public-safe. Choose a separate private factory repository for sensitive run metadata; shared public factory source can remain separate.
3. Protect the default branch. Require review for changes to role authority, privacy, protected actions, PO core instructions, schemas and release policy.
4. Verify the license and contribution policy before publishing or reusing.
5. Make sure the current factory commit, policy/role versions and `docs/technical` baseline are pinned for the run.

### Product repository

1. Confirm the exact specification and acceptance criteria with the product customer. Record source path and SHA; do not copy them into a public factory journal.
2. Keep product code, tests, local integration contracts and product-specific instructions in this repository.
3. Mark which instruction files are allowlisted, who may edit them, and how a change is reviewed. Pin the current instruction SHA.
4. Decide branch/PR convention and write scopes for PO, manager and Worker roles. This is policy/review scope, not a Cloud subagent sandbox.
5. Create a disposable product repo for WP0 tests; never use production product data to prove access controls.

## Create and publish the Cloud environment

Use current ChatGPT UI labels; they can change. Official flow is generally **Work in → Cloud → Select environment → Create environment**:

1. Connect GitHub through the intended ChatGPT identity.
2. Select factory repo and target product repo. If selection of both is unavailable or the task cannot read/write both, record failure and stop this deployment path.
3. Let Codex inspect/setup the repositories and dependencies. Review every setup command and reported change before publishing.
4. Configure only required runtime versions, tools, services and package domains. Browser/computer use is currently not documented as available in Cloud tasks; do not make it a critical dependency.
5. Add only approved domain access. A reachable domain does not grant application permission.
6. For credentials sent to an allowed HTTPS service, use a network secret if supported; direct environment variables are visible to programs running in the task. Use disposable/scoped credentials for validation.
7. Set sharing to the intended audience. Review all prepared files and secret references before publishing.
8. Publish the environment. Run a new disposable Cloud task, verify dependencies, checkout refs, read/write permissions and network policy. Record source refs and outcomes.
9. Re-publish after setup changes and verify a new task; existing tasks retain their own state.

## WP0 validation checklist

Do not mark the setup ready until the WP0 evidence log records:

- date/time, non-identifying account/plan class, environment ID/name and selected repo refs;
- parent-child-grandchild and required full topology evidence, thread count and attempted/accepted model controls;
- observed discovery of root instructions, custom agent config and repo skills;
- permission inheritance and actual GitHub identity/write behavior per repo;
- non-sensitive network/secret behavior;
- plan usage/reset visibility, session reopen and VM retention observations;
- conclusion by capability: documented, runtime-verified, unknown or blocker.

The initial maximum of six open spawned-agent threads is a configuration target only; set a lower cap if that is all the intended Cloud runtime proves. Required A-03 hierarchy remains a pass condition; do not lower it to fit an unverified cap.

## Handoff

Once the environment is published and WP0 passes, supply the environment/repository refs and current factory/product SHAs to the run operator. Do not put credentials in the handoff. Continue with [production runbook](production-runbook.md).

