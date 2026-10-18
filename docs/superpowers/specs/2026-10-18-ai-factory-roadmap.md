# AI Factory 1.0 — дорожная карта и оценка

**Статус:** план полной версии 1.0 для Codex Cloud через ChatGPT-подписку; не разрешение начать реализацию
**Связанный дизайн:** [`2026-10-18-ai-factory-design.md`](2026-10-18-ai-factory-design.md)
**Источник требований:** [`docs/specification.md`](../../specification.md)

## План релиза

Все пакеты ниже входят в полную 1.0. Последовательность уменьшает технический риск; она не разрешает выпускать сокращённую версию без всех требований и сценариев A-01–A-15.

| WP | Содержание / результат | Зависимости | Оценка (инженер-недели) | Главные риски | Проверка выхода |
|---|---|---|---:|---|---|
| WP0 | Codex Cloud feasibility: eligible account/plan, one environment with factory+product repos, Cloud task start/continue, subagent/model/tool support inside Cloud, GitHub writes/PRs to each repo, network secrets, usage dashboard and reset/limit behavior | — | 2–3 | Codex Cloud subagent and dual-repo workflow may be unavailable; usage may not be attributable per run; no API fallback allowed | Run hands-on Cloud-only spike under the intended ChatGPT account; record screenshots/refs and every missing capability; obtain human decision for any 1.0 blocker |
| WP1 | Schemas, IDs, authority policy, run/work-item FSM, append-only event conventions, privacy classes, redaction rules, subscription-usage ledger with explicit unknowns | WP0 (для Cloud-specific fields) | 2–4 | Parallel writers and schema versioning; usage evidence is account-level | Schema review; reconstruct active/paused run from Git files alone; distinguish plan dashboard evidence from estimates |
| WP2 | Connect selected GitHub repos in Cloud environment; independent factory/product refs; branches/PRs; two-repo saga, idempotency and reconciliation | WP1 | 3–5 | Environment may have limited multi-repo ergonomics; partial writes and branch conflicts | Inject failure after each saga step; reconcile duplicate/conflicting commits; verify both repo histories and no history erase |
| WP3 | Published Cloud environment profile, role instructions/skills, PO/Chief/Planner/Worker workflows, dynamic org and scoped context; Cloud subagents if proven, otherwise PO-authored queue + operator-started role-specific Cloud tasks | WP0–WP2 | 4–6 | Manual launch relay reduces autonomy; separate model/tool choice or context isolation may not be supported in Cloud | Demonstrate flat, PO→Planner→Worker, and PO→Chief→2 Planners→Workers with evidence of distinct roles, scopes and Cloud tasks; if not possible, report blocker |
| WP4 | Hierarchical work/resource ledger, subscription allowance monitoring, task-level soft reservations/retry caps, pause/checkpoint/resume same Cloud task, Git-only recovery task after retention expiry | WP1–WP3 | 3–5 | No documented per-run/per-role hard cap; plan allowance is shared and task usage varies | Reach a task/window usage limit safely; pause at task boundary, inspect account usage/reset, continue same task or recover into a new task without duplicate work |
| WP5 | Local task acceptance, product PO acceptance, integration/requirement coverage, escalation and consequential human approvals | WP1–WP4 | 2–4 | Local test success mistaken for product completion | Run A-02/A-12; reject deliberately incomplete integration; verify immutable acceptance criteria |
| WP6 | Prompt improvement proposal/authority chain, experiment registry, comparable evaluation, adopt/reject/rollback and product-vs-factory classification | WP1, WP3, WP5 | 3–5 | Confounding, low sample size, prompt drift in active runs | Run A-08–A-10/A-15; demonstrate pinned old run and future run using new/rolled-back version |
| WP7 | Security hardening: least privilege, allowlisted local instructions, secret isolation, egress policy, redaction, public/private data lifecycle, protected actions | WP0–WP6 | 3–5 | Prompt injection, accidental PII/IP leakage, broad credentials | Threat-model review; hostile repo instruction tests; secret canary, redaction and publication gate checks |
| WP8 | Full E2E acceptance, multi-product reuse, reliability/recovery, observability, docs/operator runbook, independent release review | WP0–WP7 | 4–6 | Rare failure paths and missing evidence; uncertain costs | All A-01–A-15 with retained Git evidence; full P-001–P-048 audit; independent reviewer signs release report |

### Объём и календарная оценка

- Сумма прямых оценок: **26–43 инженер-недель**.
- Резерв 30–40% на Cloud capability gate, ручную координацию, интеграцию и повторное прохождение: **34–61 инженер-недель**.
- Плановая команда: 2–3 инженера ядра, part-time security/reliability reviewer, product owner/заказчик для требований и consequential approvals.
- При указанной команде: примерно **14–23 календарных недели**. Уверенность низкая (ориентировочно ±40%) до результата WP0; оценки не включают ожидание внешних доступов и плановых reset windows.
- API расходы и дополнительные credits исключены выбранной архитектурой. Текущая стоимость ChatGPT subscription считается уже предоставленным ресурсом и не включена в инженерную оценку; смена плана и доплата за flexible pricing также не предполагаются.

### Release gate

Релиз 1.0 готов только после прохождения всех WP выходных проверок, доказуемого покрытия P-001–P-048, наблюдаемых результатов A-01–A-15 и независимой проверки. Если Cloud subagents, multi-repository writes, role scoping или usage governance не подтверждены, это оформляется как блокер с вариантами и последствиями; OpenAI API не включается как автоматический fallback. Любое платформенное препятствие оформляется с фактическими источниками, вариантами и нужными capability; автоматический scope reduction запрещён. Изменение архитектурной границы или scope утверждает человек. Такая изменённая версия уже не соответствует полной 1.0, описанной в [`docs/specification.md`](../../specification.md).
