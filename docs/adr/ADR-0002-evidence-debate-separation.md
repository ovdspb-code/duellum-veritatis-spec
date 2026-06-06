# ADR-0002 — Разделение доказательства и дебатов (evidence/debate separation)

**Статус:** ACCEPTED — утверждён OD (Олег) 2026-06-06 (заменяет PROPOSED/DRAFT от
2026-06-05). Инварианты D11–D13 и D1b+ — обязательный контракт для непретроспективных
выпусков; реализованы в gate (`duellum_gate/methodology_checks.py`) и схемах
`docs/schemas/`.
**Дата:** 2026-06-05 (PROPOSED) → 2026-06-06 (ACCEPTED)
**Decision owner:** OD / Олег
**Дополняет:** [`ADR-0001-integrity-architecture.md`](ADR-0001-integrity-architecture.md) (ACCEPTED) — не заменяет его инварианты D1–D10, а добавляет D11–D13 и усиливает D1b.
**Контекст:** внешний аудит выпуска No.3 нашёл общий failure mode — дебаты, compute и принятая доказательная база сливались в один поток, и дайджест заявлял больше, чем показал тест (toy/proxy выдан за результат уровня статьи).

## 0. Решение в одном абзаце

Публичный выпуск должен поставлять не только транскрипт и вердикт. Решающий
тест регистрируется ДО compute (`evidence_task`), каждый под-claim сопоставлен
своей доказательной базе с лестницей допуска accepted/rejected/quarantined
(`claim_evidence_matrix`), вердикт структурирован с явными «что подтверждает /
что НЕ подтверждает» и разрешённой/запрещённой формулировкой
(`arbiter_report`), а реально проведённый order-swap имеет независимо
проверяемый артефакт с хэшами (`order_swap_audit`). Жанр выпуска фиксируется
полем `epistemic_status`, и toy/proxy-решающее наблюдаемое не может обосновывать
`scientific_result`.

## 1. Контекст и проблема

ADR-0001 закрепил независимого слепого арбитра, рандомизацию сторон, real
compute для числовых claim'ов и release gate. Аудит No.3 показал остаточный
зазор: gate проверял форму выпуска, но не отделял **слой доказательства** от
**слоя дебатов**. В результате:

- решающее наблюдаемое (observable) выбиралось и описывалось постфактум внутри
  прогона, а не регистрировалось заранее;
- toy/proxy-статистика (Gram-overlap) была представлена как findings уровня
  seed-статьи, хотя не реализовывала наблюдаемое статьи (paper-faithful);
- не было машинно-читаемой карты «какой под-claim какой evidence убит/выжил/
  только спекуляция»;
- вердикт арбитра не фиксировал разрешённые/запрещённые публичные формулировки,
  поэтому дайджест мог тихо переусиливать claim;
- order-swap (D1b) сообщался как голый статус в sidecar без независимо
  проверяемого артефакта с хэшами входа/выхода.

## 2. Non-goals

Этот ADR НЕ вводит peer review, не назначает claim tiers, не делает обязательным
запуск paper-faithful compute для каждого выпуска и не продвигает claim в
discovery/manuscript status. Он добавляет минимальный контракт честности между
доказательством и дебатами.

## 3. Decision records / invariants

### D11. Pre-registered evidence task; toy/proxy не обосновывает scientific_result

Для числового / real_compute / mixed выпуска решающий тест регистрируется ДО
compute в `evidence_task.json`. Минимально: frozen claim + `frozen_at`,
`registered_before_compute=true`, `observable` (имя, точное определение,
`paper_faithful` флаг, `evidence_basis`), comparator ladder (минимум structured
vs matched-random), заранее зафиксированные thresholds с обоснованием, metrics с
вариантами (raw/folded/signed/directional), seeds (count+level), null-модели,
failure modes, критерии принятия/отклонения evidence, флаг `decisive`.

Если решающее наблюдаемое НЕ paper-faithful (toy/proxy), выпуск может быть
опубликован как `scout` / `protocol_pilot` / `methodological_case_study`, но
**не** как `scientific_result`.

`observable.evidence_basis` фиксирует, ЧЕМ установлено решающее наблюдаемое:
`simulation` (только конечно-размерная численная модель) / `semi_analytic`
(аналитика с численной сверкой, напр. DMFT, проверенный симуляцией) / `analytic`
(замкнутая форма / предел) / `empirical_data` (проверка на реальных данных
системы). paper-faithful наблюдаемое, измеренное ТОЛЬКО симуляцией, — это
результат уровня модели, а не находка уровня статьи: `scientific_result` требует
базис выше голой `simulation` (semi_analytic / analytic / empirical_data); иначе
выпуск максимум `methodological_case_study`. Так закрывается остаточный зазор:
`paper_faithful=true` сам по себе не делает finite-N симуляцию находкой уровня
статьи.

Схема: `docs/schemas/evidence_task.schema.json`.

### D12. Claim/evidence matrix с лестницей допуска

Каждый непретроспективный выпуск поставляет `claim_evidence_matrix.json`: по
строке на под-claim, с полями evidence, diagnosticity (high/medium/low/none),
direction (supports/contradicts/neutral/untested), scope (точный режим/модель,
в котором строка верна), evidence_status (accepted/rejected/quarantined/
not_applicable), claim_status (supported/contradicted/unsupported/speculative).
`unsupported` означает «claim сделан, но не подкреплён» — маркер overclaim-риска.

Схема: `docs/schemas/claim_evidence_matrix.schema.json`.

### D13. Структурированный отчёт арбитра

Каждый непретроспективный выпуск поставляет `arbiter_report.json`:
`outcome_classification` (тот же enum, что D4), evidence_quality, protocol_conduct,
непустые списки `what_it_supports` / `what_it_does_not_support`,
`permitted_wording` / `forbidden_wording`, и `confidence_and_limitations`.
`outcome_classification` обязан совпадать с `outcome` в sidecar — расхождение
блокирует публикацию. Так дайджест не может тихо заявить больше, чем нашёл
арбитр.

Схема: `docs/schemas/arbiter_report.schema.json`.

### D1b+. Order-swap, который реально прогонялся, поставляет аудит-артефакт

Усиление ADR-0001 D1b: если `order_swap_audit.status ∈ {PASS, UNSTABLE}`,
выпуск обязан приложить `order_swap_audit.json` с `checked_at`, хэшами входа и
выхода обоих прогонов (natural/swapped), substantive-вердиктами обоих прогонов и
полем `stability`. `stability` обязан совпадать со статусом в sidecar. Голый
статус без проверяемого артефакта больше не принимается для реально
проведённого order-swap. NOT_RUN/WAIVED (ретро- или OD-waiver) — без изменений.

Схема: `docs/schemas/order_swap_audit.schema.json`.

### Жанр выпуска: epistemic_status

Каждый непретроспективный выпуск объявляет `epistemic_status` из набора:

```text
scout
protocol_pilot
methodological_case_study
scientific_result
```

`scientific_result` утверждает finding уровня статьи и поэтому требует
paper-faithful решающего наблюдаемого (см. D11) с `evidence_basis` выше голой
`simulation`. Решающее наблюдаемое бывает только у числового / real_compute /
mixed выпуска: conceptual / literature_only-дуэль не имеет решающего
наблюдаемого и потому не может быть `scientific_result` (gate блокирует такую
комбинацию до проверки toy/proxy). Finite-N симуляция с paper-faithful
наблюдаемым, но `evidence_basis="simulation"`, — максимум
`methodological_case_study`: gate блокирует её как `scientific_result`.

## 4. Public/private artifact boundary (дополнение к ADR-0001 §4)

Добавляются к публичным артефактам непретроспективного выпуска:

```text
evidence_task.json          (numeric/real_compute/mixed)
claim_evidence_matrix.json  (каждый выпуск)
arbiter_report.json         (каждый выпуск)
order_swap_audit.json       (если order-swap реально прогонялся)
```

Эти артефакты — машинно-читаемый доказательный слой; нормативный источник —
схемы в `docs/schemas/`, шаблоны в `docs/templates/`.

## 5. Scope и retrospective carve-out

D11–D13 и D1b+ применяются к непретроспективным выпускам. Ретроспективные
pre-D1b выпуски (#1/#2) с `order_swap_audit.waiver_reason =
retrospective_release_before_D1b` полностью освобождены от слоя методологии — они
предшествуют контракту и опубликованы под уже принятой ретро-оговоркой.

Ретро-оговорка привязана к дате (см. ADR-0001 D2, `D1B_CUTOVER = 2026-06-05`):
освобождение действует, только если `date` выпуска строго раньше cutover. Более
поздний выпуск не может выйти из методического слоя, скопировав строку waiver —
гейт (`common.is_retro_release`) такую комбинацию не признаёт ретро и прогоняет
полный контракт.

## 6. Enforcement

Реализовано в `dispute-digest/duellum_gate/methodology_checks.py`, вызывается из
`integrity_gate.gate()` после core-проверок; артефакты валидируются против схем
(`schema_checks.validate_against_schema`, fail-closed). Тесты:
`dispute-digest/test_phase7.py`. Реальный публикационный путь
(`auto_publish.py`) вызывает `gate()` с сырым sidecar, поэтому контракт
действует на все непретроспективные публикации (ADR ACCEPTED 2026-06-06).

Публикация артефактов: `publish_digest.py` принимает `--methodology-artifact
FIELD=PATH` (повторяемый) и копирует каждый артефакт в публичный bundle как
`disputes/{slug}.{field}.json`, добавляет его в коммит и в запись
`disputes.json` под ключом `methodology`. Пути проходят через D8 filename-guard.
`auto_publish.py` резолвит sidecar-указатели (`claim_evidence_matrix`,
`arbiter_report`, `evidence_task`, `order_swap_audit_file`) и передаёт их.
Доказательный слой теперь едет вместе с релизом, а не только на входе гейта.

## 7. Acceptance criteria для перевода ADR-0002 в ACCEPTED — все выполнены (2026-06-06)

1. ~~OD подтверждает набор `epistemic_status` и правило paper-faithful → scientific_result.~~ — подтверждено OD 2026-06-06.
2. ~~`publish_digest.py` копирует методологические артефакты в публичный bundle.~~ — сделано 2026-06-05.
3. ~~FORMAT.md §9a (методологические артефакты) согласован с этим ADR и схемами.~~ — сделано.
4. ~~Полный тест-suite (phase0–7) зелёный.~~ — сделано.

## 8. References

См. ADR-0001 §7. Дополнительно мотивирующий источник — внешний аудит выпуска
No.3 (toy-proxy overclaim) и seed-статья дуэли No.3 (arXiv:2605.26551).
