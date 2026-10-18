# AI Factory 1.0 — дорожная карта и оценка

**Статус:** план полной версии 1.0; не разрешение начать реализацию
**Связанный дизайн:** [`2026-10-18-ai-factory-design.md`](2026-10-18-ai-factory-design.md)
**Источник требований:** [`docs/specification.md`](../../specification.md)

## План релиза

Все пакеты ниже входят в полную 1.0. Последовательность уменьшает технический риск; она не разрешает выпускать сокращённую версию без всех требований и сценариев A-01–A-15.

| WP | Содержание / результат | Зависимости | Оценка (инженер-недели) | Главные риски | Проверка выхода |
|---|---|---|---:|---|---|
| WP0 | Feasibility spike: tenant entitlement, managed session lifecycle, subagents, role/model/tool controls, GitHub access, secret path, cancellation, spend semantics, artifact recovery | — | 1–2 | Обязательный capability требует запрещённого постоянно работающего сервера; quota accounting неполон | Заполнена таблица capability evidence; один облачный end-to-end proof-of-concept; blockers/решения оформлены до архитектурной фиксации |
| WP1 | Schemas, IDs, authority policy, run/work-item FSM, append-only event conventions, privacy classes, redaction rules | WP0 (для провайдерских полей) | 2–4 | Параллельные записи и версия схем | Schema review; round-trip reconstruct active/paused run from repository files |
| WP2 | GitHub App/repo access; independent factory/product refs; branches/PRs; two-repo saga, idempotency and reconciliation | WP1 | 3–5 | Partial writes, branch conflicts, permission mismatch | Inject failure after each saga step; reconcile duplicate and conflicting commits; verify no history erase |
| WP3 | Managed runtime adapter, PO/Chief/Planner/Worker invocation, dynamic org topologies, scoped context and recursive task delegation | WP0–WP2 | 3–5 | Subagent model/tool inheritance prevents least privilege; concurrency/provider limits | Demonstrate flat, PO→Planner→Worker, PO→Chief→2 Planners→Workers; verify role/context evidence |
| WP4 | Hierarchical budget/reservation ledger, retry caps, pause/checkpoint/resume with new cloud session, provider error classification | WP1–WP3 | 2–4 | Provider usage null/late; spend control coverage unclear | Force quota/window pause, resume without duplicated task, prove cap behavior with reliable observed amounts or explicit unknowns |
| WP5 | Local task acceptance, product PO acceptance, integration/requirement coverage, escalation and consequential human approvals | WP1–WP4 | 2–4 | Local test success mistaken for product completion | Run A-02/A-12; reject deliberately incomplete integration; verify immutable acceptance criteria |
| WP6 | Prompt improvement proposal/authority chain, experiment registry, comparable evaluation, adopt/reject/rollback and product-vs-factory classification | WP1, WP3, WP5 | 3–5 | Confounding, low sample size, prompt drift in active runs | Run A-08–A-10/A-15; demonstrate pinned old run and future run using new/rolled-back version |
| WP7 | Security hardening: least privilege, allowlisted local instructions, secret isolation, egress policy, redaction, public/private data lifecycle, protected actions | WP0–WP6 | 3–5 | Prompt injection, accidental PII/IP leakage, broad credentials | Threat-model review; hostile repo instruction tests; secret canary, redaction and publication gate checks |
| WP8 | Full E2E acceptance, multi-product reuse, reliability/recovery, observability, docs/operator runbook, independent release review | WP0–WP7 | 4–6 | Rare failure paths and missing evidence; uncertain costs | All A-01–A-15 with retained Git evidence; full P-001–P-048 audit; independent reviewer signs release report |

### Объём и календарная оценка

- Сумма прямых оценок: **23–40 инженер-недель**.
- Резерв 25–40% на платформенную неопределённость, интеграцию и повторное прохождение: **29–56 инженер-недель**.
- Плановая команда: 2–3 инженера ядра, part-time security/reliability reviewer, product owner/заказчик для требований и consequential approvals.
- При указанной команде: примерно **12–20 календарных недель**. Уверенность низкая (ориентировочно ±40%) до результата WP0; оценки не включают ожидание внешних доступов, закупки, провайдерские квоты и тарифы.
- Cloud/API расходы не оценены без проверенного тарифа и attribution. Любая покупка требует заранее полученного разрешения.

### Release gate

Релиз 1.0 готов только после прохождения всех WP выходных проверок, доказуемого покрытия P-001–P-048, наблюдаемых результатов A-01–A-15 и независимой проверки. Любое платформенное препятствие оформляется с фактическими источниками, вариантами, нужными capability и последствиями; автоматический scope reduction запрещён. Утверждать исключение/изменение scope вправе человек, но такая версия уже не соответствует полной 1.0, описанной в [`docs/specification.md`](../../specification.md).
