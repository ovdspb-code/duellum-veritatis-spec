# ADR-0001 — Архитектура целостности Duellum Veritatis

**Статус:** ACCEPTED — утверждён OD (Олег) 2026-06-04 (заменяет DRAFT/ПРЕДЛОЖЕНО от той же даты)  
**Дата:** 2026-06-04  
**Decision owner:** OD / Олег  
**Открытые операционные follow-up'ы (не блокируют ACCEPTED, ведутся отдельно):** ретро-дисклеймер для уже опубликованных #1/#2 (§6 п.9); пометка старых first-version документов как `SUPERSEDED`, если хранятся рядом (§6 п.8). Enforcement новых полей (§6 п.10) закрыт — gate исполняемый.  
**Контекст принципов:** `docs/MANIFESTO.md`  
**Операционные документы:** `docs/FORMAT.md`, `docs/PUBLIC_RELEASE_CHECKLIST.md`  
**Заменяет:** публичное использование старой первой версии M-DISPUT/controller-driven platform для серии Duellum Veritatis  
**Не заменяет:** внутренние инженерные архивы; они остаются историческими материалами, но не являются current state

## 0. Решение в одном абзаце

Duellum Veritatis публикует не «AI peer review» и не доказательства, а проверяемые состязательные AI stress-tests frozen scientific claims. Целостность выпуска обеспечивается десятью инвариантами D1–D10: независимый слепой арбитр, randomized side assignment, раздельный ledger идей и движков, валидная ничья, append-only публикация с errata, real compute для числовых claim’ов, prior-work check, безопасное обращение с чужими текстами, калибровочный журнал без ложной статистики и release gate, который не повышает статус claim’а.

## 1. Контекст

Серия состоит из публичных научных поединков: две AI-стороны под публичными именами **Атос** и **Арамис** спорят об одном frozen claim; независимый AI-арбитр выдаёт структурный verdict; числовая часть claim’а проверяется воспроизводимым compute bundle.

Поле вокруг проекта — AI debate, LLM-as-judge, автоматизированное рецензирование, model leaderboards и autonomous science agents — методологически полезно, но содержит типовые failure modes:

- debate может помогать судье при определённых предпосылках, но сам по себе не является доказательством;
- LLM-as-judge подвержен position-, verbosity-, familiarity- и self-preference-bias;
- AI-generated/AI-assisted peer review уже требует отдельной честности и маркировки;
- hidden prompt injection в manuscripts является реальным риском для LLM-assisted reviewing;
- model leaderboards подвержены selective disclosure, overfitting и Goodhart pressure;
- autonomous-science pipelines могут генерировать слабые novelty checks, flawed experiments и hallucinated numerical results.

Этот ADR превращает эти уроки в минимальные инварианты проекта. Цель — не enterprise-бюрократия, а ровно столько процедуры, сколько нужно, чтобы публичный выпуск не превращался в театральную победу убедительного текста.

## 2. Non-goals

Этот ADR не вводит:

- controller-driven platform как текущий публичный state;
- многоступенчатую FSM;
- sealed priors как обязательный публичный механизм текущей серии;
- claim tiers Tier 0–4;
- OD disposition enum;
- автоматическую модельную лигу;
- замену peer review;
- автоматическое продвижение claim’а в discovery / manuscript / registry status.

Эти механизмы могут быть полезны в будущем инженерном продукте, но не являются текущей публичной архитектурой Duellum Veritatis.

## 3. Decision records / invariants

### D1. Арбитр вне ростера и слеп к identity

Арбитр — независимый AI-движок вне состава дуэлянтов. Он не участвует в зачёте движков и не получает на вход сведений о том, какая публичная сторона каким engine/model сыграна. Реплики маркируются нейтрально (`PRO`, `CON`) без model names, session IDs, путей sandbox, технических меток или стилевых шапок, раскрывающих происхождение текста.

Конкретный arbiter engine, версия, prompt hash и input/output hashes живут в приватном `run_manifest.private.json`, а не в публичном digest/metadata. Публичный слой может сообщать только:

```json
{
  "arbiter_identity_public": "independent_ai_arbiter",
  "outside_duel_roster": true,
  "blind_to_model_identity": true
}
```

#### D1a. Нормализация входа арбитра

Перед арбитражем удаляются:

- имена конкретных моделей;
- публичные имена дуэлянтов, если они могут стать стилевым якорем;
- session IDs и технические метаданные;
- tool-paths, sandbox paths, relay names;
- декоративные заголовки, по которым можно угадать engine или сторону.

Арбитр получает только нормализованные аргументы и evidence bundle.

#### D1b. Order-swap audit

Минимальный anti-bias check: арбитраж прогоняется в двух порядках предъявления:

```text
canonical: PRO → CON
swapped:   CON → PRO
```

Если итоговый verdict меняется только из-за перестановки порядка, это не отдельная научная победа и не новый `outcome`. Это diagnostic flag:

```json
{
  "outcome": "DRAW_UNRESOLVED",
  "arbitration_status": "ARBITRATION_UNSTABLE"
}
```

**Transitional rule before ADR ACCEPTED:** `order_swap_audit = NOT_RUN` допустим только для ретро-выпусков или экспериментальных выпусков с явным `waiver_reason` и OD approval в metadata.  
**Rule after ADR ACCEPTED:** для новых публичных выпусков order-swap audit является publication blocker, если OD отдельно не записал одноразовый waiver.

#### D1c. Verbosity-confound warning

Если финальный bundle одной стороны превышает другую более чем на заданный порог, выпуск получает флаг:

```json
"verbosity_confound_warning": true
```

Порог по умолчанию: `>25%` по token count или word count. Порог можно изменить только через явную правку FORMAT/checklist, не по ходу конкретного спора.

### D2. Сторона рандомизируется; engine league требует side balance

PRO/CON назначаются случайно и фиксируются до старта раунда. Минимальная запись:

```json
{
  "side_assignment_seed_hash": "sha256:...",
  "assignment_frozen_at": "YYYY-MM-DDTHH:MM:SSZ"
}
```

Цель — отделить качество engine от удачи со стороной. Engine league не запускается до накопления side-balanced прогонов, где каждый engine в сопоставимых темах играл и PRO, и CON.

Null/absent seed-hash допустим только для ретроспективных выпусков (#1/#2),
предшествующих D1b. Ретро-оговорка привязана к **дате выпуска**: `waiver_reason =
retrospective_release_before_D1b` принимается гейтом, только если `date` выпуска
строго раньше даты ввода D1b (`D1B_CUTOVER = 2026-06-05`, первый выпуск под
полным контрактом). Более поздний выпуск не может выйти из контракта, просто
скопировав строку waiver — гейт его блокирует (fail-closed: отсутствующая дата =
не ретро). Уточнено 2026-06-05 по решению OD.

### D3. Два раздельных зачёта

**Ledger идей** фиксирует claim-level outcome:

```text
claim → evidence/repro scope → outcome → allowed wording → errata state
```

**Ledger движков** фиксирует persuasive performance under protocol:

```text
engine → side → topic class → arbiter result → arbitration status
```

Эти ledger’ы не смешиваются. Сила спорщика не доказывает истинность позиции. Победа стороны в одном выпуске не является model ranking claim.

### D4. Вердикт допускает ничью и protocol-invalid outcome

Публичный release outcome enum:

```text
PRO
CON
DRAW_UNRESOLVED
INVALID_PROTOCOL
```

Семантика:

| Outcome | Meaning | Forbidden inference |
|---|---|---|
| `PRO` | В рамках данного evidence/protocol scope арбитр счёл позицию PRO сильнее. | «Claim доказан вообще». |
| `CON` | В рамках данного evidence/protocol scope арбитр счёл атаку CON сильнее. | «Claim опровергнут во всех смыслах». |
| `DRAW_UNRESOLVED` | Evidence/protocol не развёл стороны или arbitration unstable. | «Формат провалился». |
| `INVALID_PROTOCOL` | Нарушение процедуры делает выпуск непригодным для substantive verdict. | «Claim ложен» или «claim истинен». |

Диагностические поля отделяются от outcome:

```json
{
  "outcome": "PRO|CON|DRAW_UNRESOLVED|INVALID_PROTOCOL",
  "arbitration_status": "STABLE|ARBITRATION_UNSTABLE|NOT_RUN|WAIVED",
  "protocol_status": "PASS|PARTIAL|FAIL"
}
```

`ARBITRATION_UNSTABLE` — не outcome, а arbitration status. `INVALID_PROTOCOL` — outcome только для публикации/реестра, а не научная позиция по claim’у.

### D5. Публичность append-only, с errata вместо тихой правки

Опубликованные выпуски попадают в публичный реестр append-only. Запрещено:

- тихо удалить выпуск;
- тихо заменить число;
- тихо переписать evidence/repro bundle;
- тихо изменить outcome;
- скрыть неудачный выпуск ради красивой статистики.

Разрешено и обязательно при необходимости:

- `errata`;
- `redaction_note`;
- `quarantine_note`;
- замещающий repro bundle с новым hash manifest;
- public note, что именно изменилось и почему.

Основания для redaction/quarantine:

```text
legal
privacy
security
data_provenance_failure
prompt_injection_contamination
protocol_failure
```

`disputes.json` обновляется только append-only diff. История должна быть восстановима из git или release manifests.

### D6. Числовое утверждение требует real compute; нечисловое не маскируется

Каждое числовое утверждение в публичном выпуске должно иметь:

```text
repro script
input data / data manifest
compute_manifest.json
output hashes
environment record
rerun instruction
known limitations
```

Минимальный compute bundle:

```text
repro/YYYY-MM-DD_repro.py
repro/YYYY-MM-DD_cc_data.json
repro/compute_manifest.json
```

`compute_manifest.json` должен включать:

- `script` + `script_sha256`;
- input-data paths + hashes;
- environment: Python, packages/lock file, platform;
- `random_seed` или объяснение deterministic/no-seed;
- outputs: paths, hashes, metric, value, tolerance;
- rerun instruction;
- known limitations.

Запрещено:

- вручную рисовать числа;
- выдавать proxy diagnostic за real compute;
- экстраполировать частный результат в общий закон;
- описывать conceptual/literature-only outcome как «вычисленный».

Claim type фиксируется отдельно:

```text
numeric
conceptual
literature_only
mixed
```

Basis фиксируется отдельно:

```text
real_compute
literature_review
conceptual_analysis
mixed
protocol_failure
```

### D7. Проверка новизны / prior-work check

Перед публичным выпуском создаётся `prior_work_check.md`. Минимальные поля:

```text
Claim checked
Search date
Search queries
Databases / sources
Closest prior works
What is already known
What this duel adds
What this duel does not add
Novelty status
```

По умолчанию выпуск **не делает originality claim**. Допустимые статусы:

```text
not_an_originality_claim
likely_known
unclear
candidate_new_boundary
```

Если prior-work check не выполнен, выпуск не должен использовать формулировки «новый результат», «открытие», «впервые показано».

### D8. Дуэль над чужой статьёй: isolation, screening, bounded exposure, quarantine

Когда target — опубликованная работа, pipeline обращается с ней как с недоверенными данными:

1. разбирается claim и доказательная база, не личности авторов;
2. источник цитируется корректно;
3. visible text extraction отделяется от hidden/suspicious instruction scan;
4. извлечённое представление хэшируется;
5. дуэлянтам передаются только цитируемые, явно отделённые фрагменты;
6. никакая инструкция из target text не может переопределить system/role/protocol instructions;
7. hidden prompt detection не объявляется абсолютной гарантией;
8. при обнаружении contamination выпуск получает public contamination note или уходит в quarantine.

Публичная формулировка не должна говорить «prompt injection устранён». Разрешённая формулировка:

```text
Target text was treated as untrusted data; hidden-instruction screening and bounded excerpting were applied. No absolute guarantee of removal is claimed.
```

### D9. Calibration log без circular scoring

**Two-axis verdict (OD decision 2026-06-06).** Формальный исход (`outcome` — кто
сильнее аргументировал) и `substantive_verdict` (merits-call — было ли утверждение
истинным) — это две независимые оси, и они могут законно расходиться: сторона
может выиграть спор по утверждению, истинность которого осталась неразрешённой.
Поэтому gate НЕ требует, чтобы `substantive_verdict` совпадал с победителем. При
`outcome` PRO/CON merits-call всё же обязан быть записан и быть честным: допустимы
`pro` / `con` / `unclear`; `null` и `not_applicable` для scored claim-дуэли не
честны (это уклонение от оценки по существу). Calibration log скорит именно
`substantive_verdict` против независимого later-resolution target (не против
формального исхода).

Ведётся append-only `calibration/calibration_log.jsonl`, но calibration scoring считается только против независимого later-resolution target.

Допустимые independent resolution sources:

```text
external_replication
erratum
OD_accepted_correction
later_duel_resolution
independent_human_review
```

Если такого target’а нет, запись остаётся:

```json
"calibration_target_status": "pending"
```

Brier/log score нельзя считать против собственного verdict арбитра: это circular calibration. До достаточного N публичная страница calibration должна говорить:

```text
Calibration is being accumulated; no statistical calibration claim is made yet.
```

### D10. Publication gate; publicity does not promote the claim

Публикация проходит через явный gate:

```text
PUBLIC_RELEASE_CHECKLIST → approved sidecar → publish step → append-only registry
```

Release gate подтверждает только, что выпуск прошёл минимальные требования публикации. Он не означает:

- claim доказан;
- claim стал discovery;
- claim готов к paper/manuscript/product wording;
- engine league может быть обновлена;
- OD принял substantive claim movement.

Любое усиление wording claim’а требует отдельного OD decision и отдельного документа/записи.

## 4. Public/private artifact boundary

### Public artifacts

```text
index.html
digest.pdf
metadata.json
prior_work_check.md
repro/compute_manifest.json
repro/script and data when applicable
errata/errata.jsonl when applicable
```

### Private/internal artifacts

```text
run_manifest.private.json
model identities
arbiter engine/version
session IDs
sandbox paths
full raw transcripts if they contain technical noise or sensitive content
unredacted third-party extraction diagnostics
```

Public docs may include public document version/hash. They must not include model identities or session-level execution details.

## 5. Consequences

### Benefits

- Integrity is a release property, not a vibe.
- DRAW and INVALID_PROTOCOL reduce pressure to fabricate a winner.
- Real compute prevents numerical theater.
- Private run manifests preserve reproducibility without leaking model/session noise.
- D8 reduces prompt-injection risk without pretending to eliminate it.
- Calibration log avoids false precision until independent targets exist.

### Costs

- D1/D1b add at least one extra arbiter run per new public duel.
- D6 requires manual or scripted manifest work for numerical releases.
- D7 adds literature/source discipline before publication.
- D8 requires untrusted-document handling: extraction, hidden-instruction screening, hashing, bounded excerpting, contamination note/quarantine.
- D9 requires delayed longitudinal maintenance and cannot produce early impressive charts.
- PUBLIC_RELEASE_CHECKLIST is manual until tooling catches up.

The cost is acceptable for a small public series because the package remains a checklist + manifests, not a full controller/FSM platform.

## 6. Acceptance criteria — satisfied at ACCEPTED (OD, 2026-06-04)

These were the gating criteria for ACCEPTED status. Items 1–7 were satisfied at
accept; items 8–10 are recorded operational follow-ups that do not block
ACCEPTED and are tracked in the header. This section is retained as the
acceptance record, not as a pending checklist.

1. ✅ `MANIFESTO.md`, `FORMAT.md`, `PUBLIC_RELEASE_CHECKLIST.md`, and this ADR use the same outcome enum.
2. ✅ `ARBITRATION_UNSTABLE` is consistently an arbitration status, not an outcome.
3. ✅ `order_swap_audit` has a clear publication-blocker or waiver rule.
4. ✅ `compute_manifest.json` and public `metadata.json` templates exist.
5. ✅ D8 uses isolation/screening/quarantine language everywhere; no promise of complete prompt-injection removal remains.
6. ✅ D9 states that calibration scoring requires independent later resolution.
7. ✅ Cited evidence has exact bibliographic links or the factual claim is removed/softened.
8. ⏳ Operational follow-up: old first-version documents are marked `SUPERSEDED` if stored near the public package.
9. ⏳ Operational follow-up: existing releases #1/#2, if publicly visible, receive a retroactive disclaimer and metadata note.
10. ✅ `auto_publish.py` / `publish_digest.py` enforce the new fields (the integrity gate is now executable; the formerly-recorded gap is closed).

## 7. References

> Все arXiv-ID и DOI ниже сверены с источником 2026-06-04 (Atos): тайтлы совпадают,
> ссылки резолвятся. Источник про долю AI-рецензий (Shen & Wang) использовать как
> detector-classified/suggestive, не как ground truth.

### Methodological anchors

These works motivate design choices but do not by themselves validate Duellum Veritatis as scientific proof.

- Geoffrey Irving, Paul Christiano, Dario Amodei. “AI safety via debate.” arXiv:1805.00899, 2018. <https://arxiv.org/abs/1805.00899>
- Akbir Khan, John Hughes, Dan Valentine, Laura Ruis, Kshitij Sachan, Ansh Radhakrishnan, Edward Grefenstette, Samuel R. Bowman, Tim Rocktäschel, Ethan Perez. “Debating with More Persuasive LLMs Leads to More Truthful Answers.” arXiv:2402.06782, 2024. <https://arxiv.org/abs/2402.06782>
- Glenn W. Brier. “Verification of Forecasts Expressed in Terms of Probability.” *Monthly Weather Review*, 1950.
- Tilmann Gneiting, Adrian E. Raftery. “Strictly Proper Scoring Rules, Prediction, and Estimation.” *Journal of the American Statistical Association*, 2007. DOI: 10.1198/016214506000001437.

### Cited evidence / risk anchors

These sources support factual risk statements used in the ADR.

- Lianmin Zheng et al. “Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena.” arXiv:2306.05685, 2023. <https://arxiv.org/abs/2306.05685>
- Peiyi Wang et al. “Large Language Models are not Fair Evaluators.” arXiv:2305.17926, 2023. <https://arxiv.org/abs/2305.17926>
- Lin Shi et al. “Judging the Judges: A Systematic Study of Position Bias in LLM-as-a-Judge.” arXiv:2406.07791, 2024. <https://arxiv.org/abs/2406.07791>
- Koki Wataoka, Tsubasa Takahashi, Ryokan Ri. “Self-Preference Bias in LLM-as-a-Judge.” arXiv:2410.21819, 2024. <https://arxiv.org/abs/2410.21819>
- Siyuan Shen, Kai Wang. “Detecting AI-Generated Content in Academic Peer Reviews.” arXiv:2602.00319, 2026. <https://arxiv.org/abs/2602.00319>  
  Use as detector-classified/suggestive evidence, not ground truth.
- Zhicheng Lin. “Hidden Prompts in Manuscripts Exploit AI-Assisted Peer Review.” arXiv:2507.06185, 2025. <https://arxiv.org/abs/2507.06185>
- Panagiotis Theocharopoulos, Ajinkya Kulkarni, Mathew Magimai.-Doss. “Multilingual Hidden Prompt Injection Attacks on LLM-Based Academic Reviewing.” arXiv:2512.23684, 2025. <https://arxiv.org/abs/2512.23684>
- Toby Murray. “PhantomLint: Principled Detection of Hidden LLM Prompts in Structured Documents.” arXiv:2508.17884, 2025. <https://arxiv.org/abs/2508.17884>
- Chris Lu, Cong Lu, Robert Tjarko Lange, Jakob Foerster, Jeff Clune, David Ha. “The AI Scientist: Towards Fully Automated Open-Ended Scientific Discovery.” arXiv:2408.06292, 2024. <https://arxiv.org/abs/2408.06292>
- Joeran Beel, Min-Yen Kan, Moritz Baumgart. “Evaluating Sakana’s AI Scientist for Autonomous Research.” arXiv:2502.14297, 2025. <https://arxiv.org/abs/2502.14297>
- Shivalika Singh et al. “The Leaderboard Illusion.” arXiv:2504.20879, 2025. <https://arxiv.org/abs/2504.20879>
