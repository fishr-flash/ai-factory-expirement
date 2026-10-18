# Трассируемость требований и приёмочных сценариев

Ссылки ведут к проектным механизмам. Указание «проверка» означает необходимое эксплуатационное свидетельство при реализации/приёмке, а не уже выполненный тест. Текущая документация описывает проект; сценарии A-01–A-15 пока не исполнялись.

## P-001–P-048

| Требование | Раздел проекта | Проектный механизм и будущая проверка |
|---|---|---|
| P-001 | [Архитектура §2–3](architecture.md#2-компоненты-и-потоки) | Два независимых GitHub repo; A-01 проверит повторное подключение фабрики к разным продуктам. |
| P-002 | [Архитектура §2](architecture.md#2-компоненты-и-потоки), [Контракты §2](contracts.md#2-манифест-запуска) | Роли, навыки, run records, ресурсы и эволюция в фабрике; product source остаётся отдельно. |
| P-003 | [Архитектура §3](architecture.md#3-контекст-и-владение) | Спецификация, критерии, локальные инструкции, исходники и продуктовая история в product repo. |
| P-004 | [Безопасность §2](security.md#2-базовые-меры), [Контракты §3](contracts.md#3-задача-и-назначение) | Read/write allowlist и минимальный контекст; проверка запрещённого доступа к нерелевантным путям. |
| P-005 | [Контракты §2](contracts.md#2-манифест-запуска), [§3](contracts.md#3-задача-и-назначение) | Стабильные ID, repo/ref/commit, role/instance, prompt versions, model и output commits; A-14. |
| P-006 | [Архитектура ADR-001/004](architecture.md#5-решения-предварительные-adr), [Процесс §8](operations.md#8-конфликты-между-репозиториями) | Идемпотентные записи и reconciliation; A-13 с реальным частичным сбое/конкуренцией. |
| P-007 | [Архитектура §3](architecture.md#3-контекст-и-владение), [Безопасность §5](security.md#5-обработка-данных-и-приватность) | Public factory, private product data не зеркалируются; A-01 и проверка утечки. |
| P-008 | [Архитектура ADR-007](architecture.md#5-решения-предварительные-adr), [Безопасность §4](security.md#4-приоритет-инструкций) | Общие роли, делегирование, ресурсы, журналы, эскалации и evolution — factory scope. |
| P-009 | [Безопасность §4](security.md#4-приоритет-инструкций), [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики) | Product-specific local instructions могут изменяться с delegated authority и evidence. |
| P-010 | [Безопасность §4](security.md#4-приоритет-инструкций) | Явный precedence chain, блокировка и escalation; A-11. |
| P-011 | [Архитектура ADR-003](architecture.md#5-решения-предварительные-adr), [Процесс §4](operations.md#4-приёмка-результатов) | Immutable requirement baseline и human-authorized amendments; тест против скрытого ослабления. |
| P-012 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики), [Контракты §5](contracts.md#5-журнал-руководителя-приёмка-и-изменение-инструкции) | Change scope `product|factory`, rationale и correct repo edits; A-15. |
| P-013 | [Архитектура ADR-002](architecture.md#5-решения-предварительные-adr) | Каждый запуск имеет PO и минимум одного Worker; A-02/A-03. |
| P-014 | [Архитектура ADR-002](architecture.md#5-решения-предварительные-adr) | PO выбирает topology по сложности, dependencies, cost, risk/resources; decision log. |
| P-015 | [Архитектура §2](architecture.md#2-компоненты-и-потоки) | Прямые Worker, Planner tier, Chief multi-direction и PO→multiple Planners включены в модель; A-02/A-03. |
| P-016 | [Процесс §2](operations.md#2-сквозной-жизненный-цикл), [Контракты §2](contracts.md#2-манифест-запуска) | Изменения топологии сохраняют задачи и attribution; лишние managers не создаются автоматически. |
| P-017 | [Процесс §1](operations.md#1-полномочия-и-контекст-ролей), [§3](operations.md#3-декомпозиция-и-контракт-назначения) | У каждого уровня ограничен контекст и authority; worker allowlist. |
| P-018 | [Контракты §3](contracts.md#3-задача-и-назначение), [Платформа §2/B-01](platform.md#2-матрица-возможностей-и-решения) | Объявленный registry, выбор роли/модели/effort, наблюдение actual model; unresolved platform gate до прототипа. |
| P-019 | [Процесс §3](operations.md#3-декомпозиция-и-контракт-назначения) | Рекурсивное разбиение с parent requirements/dependencies; A-04. |
| P-020 | [Контракты §3](contracts.md#3-задача-и-назначение) | Assignment содержит цель, пути, dependencies, artifacts, checks, resource envelope и stop conditions. |
| P-021 | [Процесс §1](operations.md#1-полномочия-и-контекст-ролей), [§4](operations.md#4-приёмка-результатов) | Worker ведёт факты и не анализирует собственную роль; A-07. |
| P-022 | [Процесс §4](operations.md#4-приёмка-результатов) | Прямой менеджер review/rework; PO — отдельная интеграционная и продуктовая приёмка. |
| P-023 | [Процесс §4](operations.md#4-приёмка-результатов) | PO владеет межкомпонентными контрактами и интеграцией. |
| P-024 | [Процесс §9](operations.md#9-эскалации) | Escalation evidence/actions/risks/decision requested; недостаток бюджета ведёт к waiting, если политика разрешает. |
| P-025 | [Процесс §4](operations.md#4-приёмка-результатов), [§6](operations.md#6-ресурсная-политика) | Ограниченные retries; одинаковый критерий не повторяется без пересмотра и ресурсов. |
| P-026 | [Процесс §6](operations.md#6-ресурсная-политика), [Контракты §6](contracts.md#6-ledger-ресурсов-и-резервирование) | Chief или PO владеет budget; единый ledger/reservations предотвращают over-allocation. |
| P-027 | [Процесс §6](operations.md#6-ресурсная-политика), [Контракты §6](contracts.md#6-ledger-ресурсов-и-резервирование) | Forecast vs observed; measured/estimated/unknown разделены. |
| P-028 | [Процесс §6](operations.md#6-ресурсная-политика), [Платформа §2](platform.md#2-матрица-возможностей-и-решения) | Provider quotas, subscription и API billing не смешиваются; extra capacity только с разрешения. |
| P-029 | [Процесс §2](operations.md#2-сквозной-жизненный-цикл), [§7](operations.md#7-остановка-и-восстановление) | `waiting` — штатное восстанавливаемое состояние, процессы могут завершиться. |
| P-030 | [Архитектура ADR-005](architecture.md#5-решения-предварительные-adr), [Процесс §7](operations.md#7-остановка-и-восстановление) | Восстановление из Git отделено от автоматического wake; A-06/A-14. |
| P-031 | [Платформа §2–4](platform.md#2-матрица-возможностей-и-решения) | Каждый provider-dependent capability связан с mechanism, fallback или явным blocker и evidence plan. |
| P-032 | [Контракты §4](contracts.md#4-фактический-журнал-исполнителя), [Архитектура ADR-004](architecture.md#5-решения-предварительные-adr) | Worker commits product artifact и factual factory log; A-02/A-07. |
| P-033 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики), [Контракты §5](contracts.md#5-журнал-руководителя-приёмка-и-изменение-инструкции) | Plans, assignments, acceptance, escalation, resource/instruction decisions — manager files. |
| P-034 | [Контракты §4](contracts.md#4-фактический-журнал-исполнителя) | Factual actions/tests/errors/retries/usage; interpretation belongs to manager. |
| P-035 | [Контракты §1, §8](contracts.md#1-общие-правила) | Scope by run/task/agent, commit refs, append-only history, redaction; A-14. |
| P-036 | [Контракты §2–6](contracts.md#2-манифест-запуска) | Machine-readable run/task/budget/assignment/config/recovery view and consistency. |
| P-037 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики) | Reflection exclusively by management; distinguish product from factory. |
| P-038 | [Процесс §1/§5](operations.md#1-полномочия-и-контекст-ролей) | No self-approval/edit of foundational prompt; manager escalates to superior; A-08. |
| P-039 | [Процесс §1](operations.md#1-полномочия-и-контекст-ролей), [Контракты §5](contracts.md#5-журнал-руководителя-приёмка-и-изменение-инструкции) | Authority matrix and superior decision; PO prompt only commissioner; A-09. |
| P-040 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики) | Task prompt changes allowed within delegated scope; base policy changes use approval record. |
| P-041 | [Контракты §3/§5](contracts.md#3-задача-и-назначение) | Effective instruction snapshot and attributable versions on assignment/change; A-09. |
| P-042 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики), [Дорожная карта §6](roadmap.md#6-оценка-эффективности-после-запуска) | Comparable task/control, quality/resource measures, regression rollback; A-10. |
| P-043 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики), [Безопасность §4–5](security.md#4-приоритет-инструкций) | Promotion needs evidence, rationale, correct repo and privacy review; A-15. |
| P-044 | [Архитектура §2/§4](architecture.md#2-компоненты-и-потоки) | Provider-hosted agent runtime; optional stateless trigger; no dedicated user server. |
| P-045 | [Архитектура §7](architecture.md#7-архитектурные-ограничения-первого-релиза), [Контракты](contracts.md) | Versioned instructions/contracts drive policy; no mandatory proprietary always-on scheduler. |
| P-046 | [Безопасность §2–4](security.md#2-базовые-меры) | Least privilege, credential scoping, non-trusted repository content, authority checks. |
| P-047 | [Безопасность §5](security.md#5-обработка-данных-и-приватность) | Redaction, private run handling, public/private evidence boundaries. |
| P-048 | [Безопасность §2](security.md#2-базовые-меры), [Архитектура §6](architecture.md#6-требует-одобрения-человека) | Explicit impact-appropriate approval for deploy/purchase/destructive/protection/publication actions. |

## A-01–A-15

| Сценарий | Проектная опора | Release evidence, необходимое для закрытия |
|---|---|---|
| A-01 | [Архитектура §2–3](architecture.md#2-компоненты-и-потоки) | Один factory config подключён к двум отдельным продуктовым репам; независимые task/run IDs, credentials и продуктовые инструкции. |
| A-02 | [Архитектура ADR-002](architecture.md#5-решения-предварительные-adr) | Запуск простой задачи без Chief/Planner; topology decision, product commit, worker log, PO acceptance. |
| A-03 | [Платформа B-01](platform.md#3-критические-неизвестные-и-blockers) | PO→Chief→2+ Planners→Workers с role/model records, scopes, per-direction accepted output и integrated stage. |
| A-04 | [Процесс §3](operations.md#3-декомпозиция-и-контракт-назначения) | Parent-child trace, сохранённые requirements/dependencies, локальная review evidence. |
| A-05 | [Контракты §6](contracts.md#6-ledger-ресурсов-и-резервирование) | Одновременные reservations не превышают лимит; usage measured или unknown; stop action записан. |
| A-06 | [Процесс §7](operations.md#7-остановка-и-восстановление), [Платформа B-03](platform.md#3-критические-неизвестные-и-blockers) | Quota-triggered waiting, checkpoint, последующая авторизованная сессия; восстановленные данные и отсутствие duplicate execution. |
| A-07 | [Контракты §4](contracts.md#4-фактический-журнал-исполнителя) | Factual worker artifacts/errors/retries/measurement status и отдельный manager analysis. |
| A-08 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики) | Связанные logs, proposal по собственным guidance Planner, решение Chief (accept/modify/reject). |
| A-09 | [Контракты §5](contracts.md#5-журнал-руководителя-приёмка-и-изменение-инструкции) | Prompt diff, attribution/authority trail, последующая задача указывает новую effective version. |
| A-10 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики) | Заранее заданный comparison/control, качество/стоимость, обоснованное keep/rollback. |
| A-11 | [Безопасность §4](security.md#4-приоритет-инструкций) | Trace эффективных правил, product-only adjustment, неизменяемые ограничения сохранены. |
| A-12 | [Процесс §4](operations.md#4-приёмка-результатов) | Интеграционные проверки и оценка внешней спецификации, формальное решение PO, не только local tests. |
| A-13 | [Процесс §8](operations.md#8-конфликты-между-репозиториями) | Индуцированный partial write/conflict разрешён reconciliation протоколом; история согласована и долговременна. |
| A-14 | [Контракты §2–8](contracts.md#2-манифест-запуска) | Независимый аудит только по GitHub files восстанавливает IDs, commits, decisions, models/prompts и next actions. |
| A-15 | [Процесс §5](operations.md#5-изменение-инструкций-и-обучение-фабрики) | Product-only вывод меняет product repo; общий вывод — factory repo; оба решения атрибутированы и обоснованы. |

## Текущая полнота доказательств

Эта таблица трассируемости показывает проектное покрытие, но не свидетельствует об эксплуатации. На момент подготовки документации ни один сценарий A-01–A-15 не запускался; все acceptance evidence остаются будущими релизными воротами. Capability-зависимые P-018, P-031 и сценарии A-03, A-05, A-06, A-13 требуют внешней верификации по [плану платформы](platform.md).
