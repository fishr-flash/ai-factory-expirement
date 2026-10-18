# Дорожная карта полного релиза 1.0

Эта дорожная карта оценивает будущую реализацию всей спецификации P-001–P-048 и демонстрацию A-01–A-15. Она не разрешает вырезать требования из релиза. Указанные этапы организуют зависимости, а не вводят меньший промежуточный критерий приёмки.

## 1. Оценочные предположения

- Один провайдерный адаптер: OpenAI Agents API и hosted sandbox.
- Один основной набор тестовых GitHub repositories и один пилотный продукт с готовыми спецификацией и проверяемыми критериями.
- Реализацию ведут 1–2 инженера по API/оркестрации, 1 инженер по agent workflow/промптам, с периодическим review безопасности и продукта; роли могут совмещаться.
- Тариф API, GitHub App и допустимые данные доступны; второй облачный провайдер, UI для широкой публики и production deployment не входят в объём.
- Неведомы стек продукта, точные лимиты и ограничения аккаунта, зрелость Agents API multi-agent для вложенных ролей и объём внешней валидации. Поэтому оценки — диапазоны до прототипов.
- Один инженерский день — 6 часов сфокусированной работы. Оценка включает дизайн уточнений, реализацию, review и проверку конкретного пакета; не включает ожидание человеком коммерческих разрешений или сроки внешней очереди провайдера.

## 2. Work packages и зависимости

| WP | Содержание и результат | Зависимости | Оценка, инженерские дни | Уверенность |
|---|---|---|---:|---|
| WP-0 Platform spike | Прототипы B-01–B-05; измеренный capability report; решение по runtime и блокерам | Одобрение тестовых аккаунтов/данных | 8–14 | Низкая до доступа к аккаунту |
| WP-1 Core contracts and repository boundary | Версионированные схемы, repo registry, run/task IDs, requirement baseline, access checks, idempotency/reconciliation | WP-0 | 7–11 | Средняя |
| WP-2 Session/runtime adapter | Создание и resume сессий, роли/модели, hosted sandbox, события, отмена, traces/usage, retries | WP-0, WP-1 | 8–14 | Низкая; зависит от B-01/B-03 |
| WP-3 Dynamic organization and planning | PO topology selection, Chief/Planner optionality, recursive decomposition, assignment contracts, authority guards | WP-1, WP-2, B-01 | 9–15 | Низкая до доказательства nested/hierarchical path |
| WP-4 GitHub access, writes and recovery | Короткоживущая auth, two-repo clone/branch/commit, PR/merge states, concurrent updates, reconciliation flow | WP-0, WP-1, WP-2, B-02 | 7–12 | Низкая до GitHub App spike |
| WP-5 Evidence, budget and resume | Factual worker/manager logs, machine-readable checkpoints, ledger, quotas/unknowns, suspend/resume, recovery commands | WP-1–WP-4 | 8–13 | Средне-низкая |
| WP-6 Governance, security and evolution | Prompt precedence, immutable constraints, approval trail, redaction, evidence-based change proposals, comparative evaluation and rollback | WP-1, WP-3–WP-5 | 8–13 | Средняя |
| WP-7 End-to-end acceptance suite and operations docs | Isolated scenario fixtures, evidence report for A-01–A-15, recovery/security runbook, public/private data handling, release decision | WP-0–WP-6 | 10–17 | Средняя |

**Суммарная первичная оценка:** 65–109 инженерских дней. Ожидаемый диапазон календарного времени при небольшой команде: 10–18 недель после решения по платформе; низкая уверенность, поскольку WP-0 может выявить необходимость пересмотра архитектуры. Добавить 20–30% резерв после spike, а не скрыто уменьшать требования. Расходы моделей, sandbox и GitHub пока **unknown**; перед каждым запуском сценариев задаётся денежный потолок человеком.

## 3. Порядок выполнения и контрольные точки

### Этап 0 — Разрешения и проверка platform fit

Владелец: technical lead + человек-заказчик. Результат: B-01–B-05 закрыты доказательствами или вынесены в решение-ready blocker с вариантами и последствиями. Проверить billing, usage, data controls, сессии, nested hierarchy, модельные настройки и GitHub App access. Стоп-критерий: обязательный сценарий не представим в облачной платформе без постоянного self-hosted orchestration.

### Этап 1 — Контракты, полномочия и межрепозиторная целостность

Владелец: systems engineer + security reviewer. Утвердить структуры `run/task/log/ledger/reconciliation`, изоляцию продуктов, полномочия и допустимую модель мержей. Небольшие contract fixtures проходят схемную проверку (проверка реализации на следующем этапе). Никакие тестовые действия не требуют доступа в приватный продукт.

### Этап 2 — Runtime и динамическое делегирование

Владелец: agent workflow engineer. Реализовать API adapter, сессионный lifecycle, PO/Chief/Planner/Worker и выбор модели/effort. Доказать все поддерживаемые топологии, рекурсивную декомпозицию, назначение и review. Любое несоответствие P-013–P-018 или A-02–A-04/A-09 — blocker, не причина подменить роль инструкцией без проверки.

### Этап 3 — GitHub интеграция и надежное состояние

Владелец: platform engineer. Двухрепозиторные ветки, credential scope, атомарность не предполагается, conflict reconciliation, factual logging и проверка восстановления из Git alone. Не мержить в protected branch автоматически, пока это отдельно не разрешено.

### Этап 4 — Бюджеты, suspension и recovery

Владелец: runtime + operations. Леджер, резервирование, квотные ошибки, расход по ролям, waiting/resume без дублей и ручной/поддерживаемый инициатор. Провести симуляции частичных записей, пропавших событий и недоступной среды.

### Этап 5 — Самокоррекция и оценка качества

Владелец: PO governance + evaluation engineer. Ролевое утверждение prompt changes, сохранение snapshots и атрибуции, сравнимые контрольные задачи, метрики и rollback. Проверить, что Worker не пишет self-review, а изменение требования невозможно без человеческого решения.

### Этап 6 — Сквозная приёмка

Владелец: human commissioner/acceptance owner. Выполнить A-01–A-15 в целевой конфигурации, собрать независимые Git evidence, platform report, security/recovery результаты и список незакрытых ограничений. Релиз допускается, только если каждый сценарий удовлетворён operational evidence; unsupported mandatory scenario блокирует релиз.

## 4. План валидации и release gates

| Gate | Проверяемое утверждение | Достаточное свидетельство |
|---|---|---|
| G0 Platform fit | Провайдер обеспечивает обязательные runtime capabilities | Повторяемый capability report из целевого API проекта и решений по всем blockers |
| G1 Repository integrity | Product/factory отделены; двусторонние commits переживают конфликты | Две независимые репы, сквозные SHA, доказанный recovery после каждого частичного исхода |
| G2 Organization | PO выбирает иерархию по делу, может добавить/убрать уровни, требования не теряются | A-02/A-03/A-04 плюс roles, parent-child и acceptance trail |
| G3 Resources/recovery | Есть справедливое совместное резервирование и resume без дубликатов | A-05/A-06/A-14, измерения и unknowns, Git checkpoint до/после |
| G4 Governance/security | Инструкции и внешние действия под надлежащей властью | A-08–A-11, security negative suite, защищённые ограничения |
| G5 Product acceptance | PO принимает целый продукт по внешним требованиям | A-12 с интеграционными свидетельствами и исходным requirement commit |
| G6 Release | Всё обязательное покрыто | Матрица A-01–A-15 с evidence refs, blockers отсутствуют, заказчик принимает решения по unresolved approvals |

## 5. Основные риски и снижение

| Риск | Вероятность/влияние | Ответ |
|---|---|---|
| У платформы нет нужной вложенной иерархии или выборочной модели на субагента | Средняя/критическое | WP-0 до реализации; отдельные sessions как кандидат, но новый controller проверить на запрет self-hosting и границы P-045. |
| Git credential попадает в код, trace или команды | Средняя/критическое | GitHub App с минимальным scope, vault/proxy spike, negative tests и ротация. |
| Sandbox/session исчезает до записи результата | Средняя/высокое | Частые Git checkpoints и новый session recovery; sandbox не считать durable. |
| Usage и квоты неизвестны/запаздывают | Высокая/высокое | Ledger с unknown, консервативные резервы и стоп, человек утверждает новый budget. |
| Параллельные направления конфликтуют в shared workspace или refs | Средняя/высокое | Ограниченные файловые области, отдельные branches, lock/reservation по файлам/компонентам, review на интеграции. |
| Эволюция инструкции подгоняет результат под шум | Средняя/высокое | Сопоставимые задачи, attribution, контрольный набор и rollback; единичное наблюдение не объявлять общим улучшением. |
| Публичный журнал раскрывает сведения о продукте | Средняя/критическое | Разделение private run state, redaction, secret scanning и human review публикации. |
| Регион/хранение провайдера не подходит продукту | Неизвестно/критическое для некоторых продуктов | До передачи данных проверить условия конкретного аккаунта; использовать synthetic fixtures до одобрения. |

## 6. Оценка эффективности после запуска

Собирать на run: долю требований с доказательствами, успех интеграционной приёмки, повторные циклы и причины, время ожидания/восстановления, измеренный и неизвестный usage, оценку/фактическую стоимость при доступных данных, число эскалаций, конфликты repo, ошибки scope, изменения prompts и качество на сопоставимых задачах. Сравнивать только классы задач с приблизительно одинаковой сложностью, моделью/инструментами и полнотой входа. Ухудшение единственного более трудного проекта не считать причинным доказательством.
