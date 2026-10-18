# Контракты и долговременные данные

## 1. Общие правила

Примеры ниже — проектные структуры, не реализация. Перед кодированием их следует оформить в машиночитаемые JSON Schema и валидировать в инициаторе и при ревью. Неизменяемые записи добавляются новыми версиями/событиями; существующий объект можно пометить superseded, но нельзя удалять или переписывать историю решения.

Каждая запись использует UTC RFC 3339 timestamp, стабильные идентификаторы, версию схемы, `run_id`, scope и ссылки на commits. В неизвестных количественных данных применяется `null` вместе с `measurement_status: unknown`, не ноль. Секреты и полные токены запрещены.

## 2. Манифест запуска

Путь: `runs/<product-key>/<run-id>/run.yaml` в репозитории фабрики.

```yaml
schema_version: "1.0"
run_id: "run-2026-10-18T080000Z-7f3a"
status: planned # planned | running | waiting | reconciliation_required | blocked | accepted | closed
created_at: "2026-10-18T08:00:00Z"
updated_at: "2026-10-18T08:00:00Z"
product:
  repository: "owner/product"
  base_ref: "main"
  specification_path: "docs/specification.md"
  specification_commit: "<sha>"
  work_branch: "ai-factory/run-..."
factory:
  repository: "owner/ai-factory"
  base_ref: "main"
  configuration_commit: "<sha>"
  effective_instruction_set: "<content-addressed id>"
provider:
  adapter: "openai-agents-api"
  session_ids: []
  environment_ids: []
  state: "not_started"
topology: "po-planner-workers"
budget_id: "budget-..."
task_index: "tasks.yaml"
next_actions: []
resumed_from: null
reconciliation:
  status: "clear"
  entries: []
```

Манифест не хранит пользовательские credentials, секретные спецификации или необработанные приватные промпты в публичной репозитории фабрики. Если сам run приватный, хранить его рабочую часть в приватной копии/приватном пространстве фабрики, но сохранить те же поля и идентификаторы.

## 3. Задача и назначение

Каждая задача — отдельный файл `tasks/<task-id>.yaml`; ссылки на `parent_id` и `requirement_ids` обеспечивают трассируемость.

```yaml
schema_version: "1.0"
run_id: "run-..."
task_id: "task-..."
parent_id: null
status: ready # ready | assigned | running | reviewing | accepted | waiting | blocked | cancelled
title: "Короткая цель"
owner_role: "planner"
assignee:
  role: "worker"
  instance_id: "agent-..."
  model: "<provider model id>"
  reasoning_effort: "medium"
  actual_model: null
  model_verification: "reported"
scope:
  objective: "Проверяемый результат"
  read_allowlist: ["src/module/**", "docs/specification.md"]
  write_allowlist: ["src/module/**", "tests/module/**"]
  forbidden: ["deployment", "credential changes"]
requirement_ids: ["REQ-...", "REQ-..."]
dependencies: []
deliverables:
  product_paths: ["src/module/file.ext", "tests/module/test.ext"]
  factory_log_path: "runs/<product>/<run>/workers/<task-id>.yaml"
acceptance_checks:
  - id: "AC-1"
    command_or_method: "document exact check"
    expected: "pass and evidence path"
resource_envelope:
  budget_id: "budget-..."
  reserved: {input_tokens: 0, output_tokens: 0, sandbox_minutes: 0, currency: "USD", amount: null}
  max_attempts: 2
stop_conditions: ["acceptance check fails twice", "quota near limit", "scope requires extra write access"]
effective_instructions:
  factory_commit: "<sha>"
  product_commit: "<sha>"
  role_prompt_version: "<sha-or-content-id>"
  task_prompt_version: "<sha-or-content-id>"
  precedence_resolution: "<record id>"
```

`reserved` — прогноз/лимит, не расход. `actual_model` и `model_verification` заполняются только доступными данными.

## 4. Фактический журнал исполнителя

Путь: `workers/<task-id>.yaml` в директории run. Должен содержать только непосредственно наблюдаемое.

```yaml
schema_version: "1.0"
run_id: "run-..."
task_id: "task-..."
agent_instance_id: "agent-..."
role: "worker"
started_at: "<timestamp>"
ended_at: "<timestamp-or-null>"
attempt: 1
instruction_snapshot: "<content-addressed id>"
actions:
  - at: "<timestamp>"
    action: "command|edit|tool_call|commit"
    summary: "Factual summary without secrets"
    result: "success|failure|unknown"
    evidence_ref: "<path or provider item id>"
checks:
  - name: "<check>"
    command: "<safe command>"
    result: "pass|fail|not_run"
    evidence_ref: "<path>"
errors: [{at: "<timestamp>", category: "<category>", summary: "<redacted factual summary>"}]
resource_observation:
  source: "agents-api-turn-usage"
  measurement_status: "measured|partial|unknown"
  input_tokens: null
  cached_input_tokens: null
  output_tokens: null
  reasoning_tokens: null
  sandbox_seconds: null
  currency: "USD"
  estimated_cost: null
artifact_commits:
  product: "<sha-or-null>"
  factory: "<sha-or-null>"
final_status: "completed|failed|waiting|cancelled"
```

Запрещены выводы вроде «модель ошиблась из-за плохого промпта», догадки о незафиксированном usage и копирование секретов из вывода.

## 5. Журнал руководителя, приёмка и изменение инструкции

Руководительская запись имеет `decision_id`, `actor`, `authority_basis`, ссылки на worker logs и commits, входные/выходные состояния, действие и rationale. Приёмка задачи указывает каждый acceptance check, его свидетельство и остаточные ограничения. Приёмка PO дополнительно ссылается на внешние требования, интеграционные проверки и версию исходной спецификации.

Изменение базовой инструкции оформляется отдельным объектом:

```yaml
change_id: "prompt-change-..."
scope: "factory|product"
target_role: "planner"
proposal_author: "agent-..."
decision_author: "agent-..."
authority_basis: "P-039 / superior role"
evidence_refs: ["<worker log>", "<task commit>"]
prior_version: "<sha>"
new_version: "<sha-or-null-if-rejected>"
decision: "accepted|modified|rejected"
expected_effect: "<measurable outcome>"
evaluation_plan: "<task class, comparison, quality/resource measures>"
rollback_condition: "<threshold>"
applies_to_runs: ["<new run IDs or future>" ]
```

Редактирование задачи не создаёт `prompt-change`, если оно не меняет базовую инструкцию роли.

## 6. Ledger ресурсов и резервирование

Объект ledger содержит `budget_id`, уполномочившего, валюту, верхний денежный лимит, временное окно, квоты провайдера (если известны), лимиты задач/ролей, операции резервирования и фактические наблюдения. Каждая операция записывается отдельным событием: `reserve`, `release`, `usage_observed`, `estimate_updated`, `limit_changed`, `stop`. Для изменения верхнего лимита требуется человеческое решение.

Сумма резервов по активным задачам не превышает доступный остаток. Измерения API помечаются источником и временем получения. Best-effort usage может быть `null` или поздно скорректирован; ledger хранит несколько измерений с `observed_at`, не переписывая прежнее. Денежная сумма рассчитывается только при известной модели тарификации/цене и помечается как estimate, пока не сверена с биллингом.

## 7. Межрепозиторная сверка

Reconciliation entry связывает `event_id`, задачу, ветки и SHA обоих репозиториев, статус каждой записи и решение. Повтор операции использует тот же idempotency key, сначала проверяет, не существует ли уже коммита/события, затем выполняет недостающий шаг. История Git не переписывается для «починки»; новая запись объясняет исправление.

## 8. Версии и эволюция схем

Схемы используют `schema_version`. Неизвестные поля при чтении игнорируются только если это безопасно; неизвестные статусы, полномочия, пути или переходы приводят к отказу закрыть задачу и ручной эскалации. Миграция данных должна сохранять исходные файлы или давать ссылку на их commit. Версия схемы и действующая конфигурация входят в checkpoint.
