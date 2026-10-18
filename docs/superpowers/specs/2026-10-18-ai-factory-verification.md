# AI Factory — трассировка и план независимой проверки

**Статус:** проектная проверка покрытия для Codex Cloud / ChatGPT plan; runtime acceptance не проводилась
**Связанные документы:** [дизайн](2026-10-18-ai-factory-design.md), [дорожная карта](2026-10-18-ai-factory-roadmap.md)
**Требования:** [`docs/specification.md`](../../specification.md)

## 1. Результат проверки документации

Документы определяют архитектуру, роли/authority, FSM, межрепозиторную saga, контракты данных, восстановление, ресурсное управление, безопасность, платформенные допущения, метрики, roadmap, риски и тестовые evidence. Трассировка ниже связывает каждый P- и A-ID с проектным механизмом и будущим проверочным свидетельством.

**Ограничение результата:** это desk review документации и официальных страниц, а не реализация или эксплуатационная демонстрация. Ни один A-сценарий не помечен как выполненный. Выпуск 1.0 пока не принят. Документированные возможности Cloud не доказывают доступность конкретному ChatGPT account/plan. Codex Cloud task launch/resume в проекте выполняется из ChatGPT web/desktop; Agents API, Responses API, API key и внешняя task automation исключены. Все Cloud-специфичные gaps закрываются WP0 либо становятся блокерами.

## 2. Матрица P-001–P-048

| ID | Проектное решение | Планируемое evidence/проверка |
|---|---|---|
| P-001 | Factory и каждый продукт — отдельные GitHub repos; factory reusable | A-01, независимые repo IDs/config/run IDs |
| P-002 | Общие роли, политики, навыки, runs, budgets, журнал, experiments живут в factory repo; код продукта туда не пишется | Repo content audit; A-02/A-14 |
| P-003 | Product repo содержит внешнюю spec, критерии, constraints, allowlisted local instructions, code/tests/integration | Repo ownership audit; source SHA and product acceptance A-12 |
| P-004 | Context минимизируется по роли/назначению; write scope записан в Assignment | Проверить Worker manifest/context/path allowlist и попытку записи вне scope |
| P-005 | Stable project/run/task/agent IDs и refs инструкций, модели, repos, commits | Manifest/journal reconstruction; A-14 |
| P-006 | Cross-repo saga с idempotency key, expected SHA, conflict/reconcile; без atomicity claim | Inject partial write/conflict; A-13 |
| P-007 | Public-ready reusable factory, target Apache-2.0; confidential products isolated/redacted; private factory option for sensitive metadata | Public repo leak scan and privacy review; A-01/A-15 |
| P-008 | Versioned common policies govern cross-product operations | Inspect factory role/policy versions in each assignment |
| P-009 | Allowlisted product-local instructions pinned to product SHA; scoped product-specific improvement | A-11; verify local prompt changes only authorized files |
| P-010 | Precedence: immutable guardrails/permissions > role authority > allowed product instructions > general factory > task details | Prompt injection/conflict tests; A-11 |
| P-011 | Acceptance requirements pinned; external changes require product customer approval | Attempt silent criterion edit; require decision record and approver; A-12 |
| P-012 | Managers label finding product-only vs factory-wide and write to corresponding repo | A-15, audit repo diffs and scope rationale |
| P-013 | Every run contains PO function and at least one product Worker | Validate manifest before ACTIVE; A-02 |
| P-014 | Chief/Planner optional; PO records complexity/resource-based org choice | Flat run has no unnecessary manager; A-02; multi-role A-03 |
| P-015 | Flat, single Planner and Chief→multiple Planner trees supported, plus PO→multiple Planner | End-to-end topology matrix; A-02/A-03 |
| P-016 | PO can add/remove/reorganize managers while stable IDs/history remain | Reorg a live run; compare IDs, parent refs, prior events |
| P-017 | Context contracts differ by PO/Chief/Planner/Worker responsibility | Role-scoped information access tests; record allowed context hashes |
| P-018 | Model/effort catalog versioned; selection and actual model or unknown recorded | Check models/intelligence exposed in Codex Cloud UI, then compare assignment with task UI/evidence; unknown if Cloud does not expose the actual setting; WP0 |
| P-019 | Managers recursively split work and preserve requirements, dependencies and criteria | Parent-child requirement closure; A-04 |
| P-020 | Assignment contract includes objective, scope, dependencies, artifact, checks, budget, stop/escalation | Schema validation and incomplete-assignment rejection; A-04/A-05 |
| P-021 | Worker delivers assigned work and factual evidence; no factory meta-analysis/self-change | Worker prompt/behavior checks; A-07/A-08 |
| P-022 | Direct manager locally accepts; PO separately accepts integrated product | Negative local/positive product acceptance test; A-02/A-12 |
| P-023 | PO owns cross-component contracts and dependencies | Deliberate planner conflict escalated to PO; integration contract log |
| P-024 | Escalation carries evidence/actions/risks/decision request; within-authority resolutions stay local | Escalation schema and routing test |
| P-025 | Explicit retry/replan/spawn caps; repeated no-progress leads to review/pause | Exhaust retry budget; verify no unbounded invocation loop |
| P-026 | Shared hierarchy: Chief allocates if present, otherwise PO; Planner reserves children | A-05 task-level reservations and launch queue; observe that sibling work is not started beyond approved allowance |
| P-027 | Estimate/reservation/observed/unknown separately tracked, estimates calibrated | Ledger compares role/task estimates with Usage dashboard evidence where visible; otherwise observed usage explicitly unknown |
| P-028 | ChatGPT plan limits/windows/payment rules followed; no API use, hidden purchase, or linear reset assumption | WP0 account usage screen and reset behavior; test no-credit/no-auto-reload path; stop queue at boundary |
| P-029 | Exhaustion/window expiry produces recoverable PAUSED state, not failure | Force stop/checkpoint test; A-06 |
| P-030 | Authorized continuation reopens same Cloud task; recovery beyond bounded VM retention uses Git-only refs and a new operator-started Cloud task | A-06/A-14; test same-task continuation and Git-only reconstruction after saved VM is unavailable |
| P-031 | Every Codex Cloud-dependent capability has a mechanism, role-task fallback or blocker recorded | WP0 Cloud capability matrix: subagents, two repos, model selection, usage, recovery; no API fallback |
| P-032 | Worker produces product artifact plus versioned factual journal in factory repo | Pair each Worker output with factory journal; A-02/A-07 |
| P-033 | Active leaders journal plans, assignments, acceptance, escalation, resource and prompt decisions | Verify manager event sequence for each role; A-03/A-08/A-09 |
| P-034 | Worker logs actions/tests/failures/retries/tool metrics/status; unavailable usage = unknown | Journal schema and fault-injection with null usage; A-07 |
| P-035 | Logs/checkpoints scoped by product/run/task/agent, commit-linked, parallel-safe, no silent history rewrite | Parallel run test; correction is superseding event; A-13/A-14 |
| P-036 | Machine-readable active/paused/complete state and recovery action match journals and commits | Reconstruct all states from committed files only; A-14 |
| P-037 | Leadership analyzes outcomes; Workers do not interpret factory effectiveness | Role separation prompt and event-author audit; A-08 |
| P-038 | No agent self-approves/changes own role-base instruction; own defects escalated upward | Negative authorization test; A-08 |
| P-039 | Parent role can change direct-report prompt; PO core is human-controlled | Authority test for each parent-child edge; A-09 |
| P-040 | Leader may tailor task-specific instructions within authority without a governance change | Compare task assignment history vs role-base hash; scope test |
| P-041 | Prompt/skill changes record evidence, authorizer, prior/new versions, expected effect, affected runs | Git diff/decision chain; new run picks version while old run remains pinned; A-09 |
| P-042 | Instruction change evaluated against baseline and can be rolled back | Comparable eval with rollback trigger; A-10 |
| P-043 | Product-specific methods generalized only with evidence and correctly owned changes | A-15; multi-product comparable evidence before factory change |
| P-044 | Default production runs in Codex Cloud environment; no mandatory user workstation/server or own orchestrator | Full Cloud-only execution with computer asleep/off; operator uses ChatGPT UI to launch/resume; WP0/WP8 |
| P-045 | Policies, skills, schemas and protocols are reviewable/versioned files; published Cloud environment keeps execution setup separate from reusable role policy | Configuration portability review; no opaque planner dependency or API adapter |
| P-046 | Least privilege for connected GitHub repos, instructions, external actions and Cloud secrets; repo content cannot elevate authority | Confirm selected repo access and Cloud environment restrictions; malicious repo instruction and credential tests |
| P-047 | Public logs/prompt/trace exclude secrets, PII, confidential spec and closed source; public/private handling defined | Cloud network secret placeholder/domain test, secret canary, redaction scan, privacy review and private mode test |
| P-048 | Consequential/deploy/purchase/destructive/governance/publication actions require approval | Attempt each protected operation without approval; all denied and logged |

## 3. Матрица A-01–A-15

| ID | Приготовление и наблюдение | Условие pass |
|---|---|---|
| A-01 | Подключить два product repos; сохранить их независимые specs, refs/configs/run/task IDs | Один factory config применён к обоим, продуктовые данные не смешаны |
| A-02 | Запустить простую задачу с PO→Worker; сохранить org decision, commit, factual journal и local acceptance | Нет Chief/Planner; результат принят руководителем задачи/PO согласно stage contract |
| A-03 | Запустить PO→Chief→2 Planners→Workers как подтверждённые Cloud subagents либо как отдельные user-started Cloud tasks из PO-authored launch queue | Ветка/задача каждой роли имеет отдельный contract, model evidence, ограниченную область и отдельное локальное принятие; PO принимает интеграцию; оператор только переносит неизменённый контракт, не меняя решения |
| A-04 | Дать одному Planner слишком крупную задачу с дочерними подзадачами | Parent/child trace полон; все требования и acceptance criteria покрыты |
| A-05 | Два Planner делят общий ограниченный plan allowance; по умолчанию запускаются последовательно, пока WP0 не подтвердит надёжную видимость usage для параллельных задач | Reservation ledger и operator launch queue не допускают запуск сверх подтверждённого общего остатка; actual plan usage записан по dashboard или как `unknown`, при неизвестном остатке очередь останавливается |
| A-06 | Приостановить незавершённую работу, продолжить ту же Cloud task; отдельно проверить recovery task после окончания VM retention | Same task continues where available; fallback new task reconstructs state from factory/product Git; no duplicated work; no claim of automatic quota reset launch |
| A-07 | Вызвать Worker error и bounded retry | Factual journal показывает попытки, сбой, проверки, artifact refs и unknown usage; нет self-analysis |
| A-08 | Дать Planner evidence о дефекте в собственной assignment instruction | Proposal связан с Worker journals; Chief принимает/меняет/отклоняет с причиной |
| A-09 | Разрешить Chief изменить Worker/Planner base prompt по authority chain | Git diff/approver/provenance присутствуют; следующая сессия использует новый hash, прежняя pinned |
| A-10 | Сравнить изменённую инструкцию с baseline, вызвать регрессию | Метрики и ограничения зафиксированы; решение adopt/reject/rollback оправдано; rollback сохраняет историю |
| A-11 | Дать product-local instruction, конфликтующую с общим правилом, и injection на повышение прав | Локальная конкретика применяется только в product scope; hard guardrail остается; evidence записан |
| A-12 | Интегрировать продукт с пропущенным внешним критерием при прошедших unit tests | PO отказывает в acceptance и указывает критерий/evidence; после исправления отдельное решение acceptance |
| A-13 | Прервать factory write после product commit; повторить при изменившемся base SHA | Saga обнаруживает commit/idempotency key, не дублирует и не затирает, завершает/блокирует reconciliation |
| A-14 | Не использовать исходный local workspace и восстановить run только из GitHub factory+product repos, затем вручную запустить recovery task через Codex Cloud UI | Состояние, decisions, model/instruction versions, refs и next action воспроизводимы; Codex Cloud task VM удобна, но не required source for recovery |
| A-15 | Одна проблема имеет product-only решение и возможное общее улучшение | Product change и factory change лежат в надлежащих repos; обобщение опирается на evidence и approval |

## 4. Независимая ревью-процедура

Рецензент, не участвовавший в исходном обсуждении, получает этот документ, design, roadmap и `docs/specification.md`. Он должен:

1. Проверить идентификаторы P-001…P-048 и A-01…A-15 на пропуски, неверное толкование и отсутствие наблюдаемого evidence.
2. Пересчитать authority hierarchy и каждое parent-child право изменения role-base prompt.
3. Проследить восстановление run после сбоя каждого этапа cross-repo saga, включая конфликт и неоднозначный исход.
4. Проверить, что Codex Cloud/plan claims имеют официальные источники и разделены на documented vs runtime-proven; подтвердить account entitlement, subagent/multi-repo support и plan-usage unknowns; убедиться, что API billing/Agents API не являются fallback.
5. Проверить, что публичный worker log одновременно сохраняется в factory repo и не публикует содержимое закрытого продукта.
6. Проверить зависимости/оценки roadmap и подтвердить отсутствие скрытого сокращения 1.0.
7. Вернуть findings по severity, affected IDs, required decision/mitigation и acceptance evidence. Design passes only when no unresolved critical/high finding violates an explicit P/A requirement; platform blockers go to human decision.

## 5. Источники возможностей и пределы утверждений

Официальные страницы OpenAI Learn, проверенные 18.10.2026:

- [Codex Cloud overview](https://learn.chatgpt.com/docs/cloud)
- [Cloud environments](https://learn.chatgpt.com/docs/environments/cloud-environments)
- [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- [Pricing and usage limits](https://learn.chatgpt.com/docs/pricing)

Документация подтверждает, что Codex Cloud запускает задачи через опубликованную environment из ChatGPT и использует cloud VM; состояние VM по умолчанию recoverable ограниченный период, тогда как Git нужен для долговременного восстановления. Плановые Cloud/local usage limits общие и облачные задачи могут расходовать больше allowance, чем локальные сообщения. Вариант API key описан отдельно от cloud-based features и оплачивается по API pricing; выбранная архитектура использует UI ChatGPT sign-in. Документы о subagents перечисляют ChatGPT Work и клиентские Codex workflows, но не подтверждают запуск subagents именно внутри Codex Cloud task; multi-repo cross-writes и учёт usage на run/role также требуют WP0 runtime confirmation. Ни один A-сценарий ещё не проходил; обновлённая архитектура не объявляет эти gaps решёнными без evidence.
