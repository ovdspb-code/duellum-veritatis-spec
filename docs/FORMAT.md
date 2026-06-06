# FORMAT — Dispute Digest publication format

**Статус:** DRAFT operational publishing spec  
**Дата:** 2026-06-04  
**Scope:** HTML+PDF digest, public metadata, evidence/repro bundle and allowed wording for Duellum Veritatis releases  
**Not scope:** internal engine/session logs, model league launch, controller/FSM design

## 1. Назначение

`FORMAT.md` задаёт публичный формат выпуска Duellum Veritatis. Главный объект выпуска — **digest + evidence/repro bundle + metadata**, а не raw transcript.

Формат должен быть читаемым для образованного неспециалиста, но не должен жертвовать методологической точностью. Мушкетёрское оформление допустимо как tone layer; оно не должно превращать выпуск в шоу или скрывать границы доказательности.

## 2. Аудитория и язык

- Основная аудитория: образованный неспециалист уровня бакалавра, не обязательно из области claim’а.
- Термины вводятся сразу через короткое пояснение / `.gloss`.
- Рабочий язык: русский; для публичного международного слоя допустим EN/RU.
- Лонгрид допустим, но целевой предел digest: 5 страниц A4, если OD не решил иначе.
- Технические детали, которые нужны для воспроизводимости, уходят в `metadata.json`, `compute_manifest.json`, `prior_work_check.md` и repro bundle.

## 3. Раскрытие формата

Серия раскрывает свою природу честно:

```text
Duellum Veritatis is a public adversarial AI stress-test of a frozen scientific claim.
```

Можно и нужно говорить:

- стороны — AI-generated / AI-assisted sides;
- публичные дуэлянты — Атос и Арамис;
- формат — состязательный stress-test / structured adversarial review;
- выпуск не является peer review, proof, endorsement или discovery claim;
- numerical parts are backed only by the attached reproducible compute bundle.

Не тащить наружу как ценность:

- concrete model names / engine versions;
- session IDs;
- sandbox paths;
- raw relay names;
- internal run identifiers;
- private prompt text, если он раскрывает модель/сессию/операционные ключи.

Допустимо публично указывать:

- public documentation version/hash;
- public protocol family (`Duellum Veritatis public stress-test protocol`);
- release package hashes;
- fact that model identities are withheld.

Имена engine/model и arbiter version живут в `run_manifest.private.json`, не в публичном digest.

## 4. Байлайн и identity

- Издатель: **Occam Research**.
- Публичный авторский псевдоним: **Hanlon Occam**.
- Реальное имя автора: только в правовом слое DOI/Zenodo, не на публичный байлайн.
- Публичные стороны: **Атос / Athos** и **Арамис / Aramis**.
- Публично не говорить «model X победила model Y».

## 5. Обязательная шапка выпуска

Шапка видна **до verdict** в каждом HTML/PDF.

Контейнер шапки несёт структурный маркер `data-duellum="status-disclaimer"`
(например `<div class="disclaimer" data-duellum="status-disclaimer">…`). Release
gate ищет именно этот атрибут, а не текст «release gate» в прозе. Маркер
позволяет менять формулировку дисклеймера, не ломая проверку, и не даёт
случайному упоминанию «release gate» в тексте сойти за дисклеймер.

Внутри шапки обязательна **gate-scope clause** с маркером
`data-duellum="gate-scope"`: явно сказано, что прохождение release gate — это
автоматические структурные проверки целостности и формата, а **не**
человеческое методологическое рецензирование, не peer review и не одобрение
существа утверждения. Гейт требует этот маркер на каждом non-retro выпуске,
чтобы читатель не прочитал «прошло гейт» как «методология одобрена».

### RU

> **Статус:** публичный состязательный AI stress-test. Это не peer review, не научное доказательство и не подтверждение тезиса. Две AI-стороны спорили о заранее зафиксированном утверждении по фиксированному протоколу; независимый AI-арбитр вынес структурный вердикт. Для числовых утверждений опубликованные числа происходят только из приложенного воспроизводимого compute bundle. Публикация означает, что выпуск прошёл release gate; она не повышает статус утверждения до открытия.

### EN

> **Status:** public adversarial AI stress-test. This is not peer review, not a scientific proof, and not an endorsement of the claim. Two AI-generated sides argued a frozen claim under a fixed protocol; an independent AI arbiter issued a structured verdict. For numerical claims, the reported numbers come only from the attached reproducible compute bundle. Publication means the duel passed the release gate; it does not promote the claim to discovery status.

## 6. Forbidden wording matrix

| Запрещено | Разрешено |
|---|---|
| «ИИ доказал, что…» | «В рамках выпуска арбитр счёл, что…» |
| «Дуэль установила истину» | «Дуэль является stress-test разбором утверждения» |
| «Claim подтверждён» | «Claim получил поддержку в рамках указанного evidence/repro scope» |
| «Claim опровергнут вообще» | «Claim не выдержал данный тест / данный scope» |
| «Модель X лучше модели Y» | «В этом выпуске публичная сторона победила по аргументации; имена движков withheld» |
| «Опубликовано — значит валидно» | «Публикация означает прохождение release gate, не повышение статуса claim’а» |
| «Prompt injection устранён» | «Target text был обработан как untrusted data; применены screening/bounded excerpting; абсолютная гарантия не заявляется» |
| «Числа иллюстративны» для публичного numeric claim | «Числа происходят из приложенного repro bundle; script/data/hashes/environment указаны в compute_manifest.json» |
| «Ничья — провал выпуска» | «DRAW_UNRESOLVED — валидный исход, если evidence/protocol не развёл стороны» |
| «ARBITRATION_UNSTABLE = победа/поражение» | «ARBITRATION_UNSTABLE — диагностический статус арбитража; outcome отдельно» |

## 7. Public metadata minimum

Каждый публичный выпуск, кроме HTML+PDF, несёт `metadata.json`.

Минимальные поля:

```json
{
  "duel_id": "dv-0001",
  "series": "Duellum Veritatis",
  "status": "public_adversarial_ai_stress_test",
  "claim": {
    "text": "...",
    "frozen_at": "YYYY-MM-DDTHH:MM:SSZ",
    "claim_type": "numeric|conceptual|literature_only|mixed"
  },
  "sides": {
    "public_names": ["Athos", "Aramis"],
    "side_assignment_seed_hash": "sha256:...",
    "model_id_public": "withheld"
  },
  "arbiter": {
    "identity_public": "independent_ai_arbiter",
    "outside_duel_roster": true,
    "blind_to_model_identity": true,
    "order_swap_audit": {
      "status": "PASS|UNSTABLE|NOT_RUN|WAIVED",
      "waiver_reason": null
    },
    "verbosity_confound_warning": false
  },
  "outcome": {
    "enum": "PRO|CON|DRAW_UNRESOLVED|INVALID_PROTOCOL",
    "basis": "real_compute|literature_review|conceptual_analysis|mixed|protocol_failure",
    "confidence": 0.0,
    "arbitration_status": "STABLE|ARBITRATION_UNSTABLE|NOT_RUN|WAIVED",
    "protocol_status": "PASS|PARTIAL|FAIL"
  },
  "reproducibility": {
    "bundle_included": true,
    "compute_manifest": "repro/compute_manifest.json",
    "rerun_status": "not_rerun|rerun_pass|rerun_fail|not_applicable"
  },
  "publication_gate": {
    "status": "PASSED|BLOCKED|WAIVED",
    "approved_sidecar_hash": "sha256:...",
    "prior_work_check": "pass|limited|fail|not_applicable",
    "third_party_text_handling": "pass|not_applicable|quarantined|contamination_note",
    "errata": []
  }
}
```

See `docs/schemas/metadata.schema.json` and `docs/templates/metadata.template.json`.

## 8. Compute manifest

For any `claim_type = numeric` or `basis = real_compute|mixed`, include `repro/compute_manifest.json`.

It must record:

- script path and hash;
- input data paths and hashes;
- Python/platform/package environment;
- seed or deterministic/no-seed note;
- output paths, hashes, metrics, values and tolerance;
- rerun instruction;
- known limitations.

All numerical values and graphs in the digest must come from the evidence/repro bundle. Manually drawn numbers are forbidden.

## 9. Prior-work check

Every release includes `prior_work_check.md`. Default stance:

```text
This release does not make an originality claim unless explicitly stated and supported.
```

Allowed novelty statuses:

```text
not_an_originality_claim
likely_known
unclear
candidate_new_boundary
```

## 9a. Methodology artifacts (ADR-0002, ACCEPTED)

> Status: ACCEPTED 2026-06-06 in [`adr/ADR-0002-evidence-debate-separation.md`](adr/ADR-0002-evidence-debate-separation.md).
> Enforced by the gate; a binding publication contract for non-retrospective releases.

Every non-retrospective release carries a machine-readable evidence layer beside
the transcript. The sidecar references each artifact by a relative path; the gate
validates it against its schema (fail-closed) and refuses to publish on mismatch.

| Artifact | Sidecar field | When required | Schema |
|---|---|---|---|
| Claim/evidence matrix | `claim_evidence_matrix` | every release | `schemas/claim_evidence_matrix.schema.json` |
| Structured arbiter report | `arbiter_report` | every release | `schemas/arbiter_report.schema.json` |
| Pre-registered evidence task | `evidence_task` | numeric / real_compute / mixed | `schemas/evidence_task.schema.json` |
| Order-swap audit | `order_swap_audit_file` | when order-swap actually ran (status PASS/UNSTABLE) | `schemas/order_swap_audit.schema.json` |

Additional sidecar fields:

- `epistemic_status`: `scout | protocol_pilot | methodological_case_study | scientific_result`.
  A decisive observable that is not paper-faithful (toy/proxy) cannot back a
  `scientific_result`.

Cross-checks the gate enforces:

- `arbiter_report.outcome_classification` must equal the sidecar `outcome`.
- `order_swap_audit.stability` must equal the sidecar `order_swap_audit.status`.

Templates for all four live in `docs/templates/`. Retrospective pre-D1b releases
(#1/#2) are exempt from this whole layer.

## 10. Page order

Digest page order:

1. Status disclaimer.
2. Frozen claim.
3. Outcome enum + arbitration/protocol status.
4. What this supports.
5. What this does **not** support.
6. Evidence/repro bundle summary.
7. Arbiter verdict.
8. Compressed PRO/CON arguments.
9. Prior-work check summary.
10. Errata / quarantine status.

Raw transcript is not the main object. It can be archived privately or separately if safe and useful.

## 11. Required digest sections

1. Ornament / series marker.
2. H1 title and short lede.
3. Mandatory status disclaimer.
4. Metadata card: date, topic, claim type, sides as Athos/Aramis, public model identities withheld.
5. Frozen claim blockquote.
6. Short explanation of claim and stakes.
7. PRO/CON cards.
8. Evidence/repro section.
9. Graphs/tables if numeric, with axes, captions and source path to outputs.
10. Arbiter verdict with outcome/basis/confidence/statuses. The two verdict axes
    are rendered as **two distinct, separately-marked blocks**: the formal
    `outcome` (who argued better) carries `data-duellum="outcome"`, the
    `substantive_verdict` (was the claim true) carries
    `data-duellum="substantive-verdict"`. When the axes diverge — a side won on
    argumentation (outcome PRO/CON) but the substantive merits-call does not
    match that direction — an explicit divergence note carrying
    `data-duellum="axis-divergence"` must say so, so a reader cannot read a won
    argument as a confirmed claim. The gate requires both axis blocks on every
    non-retro release and the divergence note whenever the axes diverge.
11. “What this supports.”
12. “What this does not support.”
13. “Where the losing side still has a point” / “remaining uncertainty.”
14. Prior-work note.
15. Appendix: compute parameters, hashes, environment, rerun command.
16. Footnotes / references.
17. Errata/quarantine note if applicable.

## 12. Numerical precision rules

- Report no more precision than the compute/output supports.
- Distinguish exact, approximate, simulation-derived and literature-derived numbers.
- Do not silently round across a threshold.
- State tolerance when a numerical threshold decides outcome.
- Do not extrapolate a local simulation to a general law.
- If graph values are derived from code, the source output file and hash must appear in `compute_manifest.json`.

Correct appendix wording:

```text
Numbers and figures come from the attached repro bundle. Script, data, hashes, environment and rerun instructions are recorded in repro/compute_manifest.json.
```

## 13. Styling

The visual style can remain recognizable, but style never outranks clarity.

Locked style defaults:

- background: `#222d49` with soft radial gradients;
- text: `#f3efe4`;
- headings: `#fbf6e8`;
- gold accent: `#f0c469`;
- PRO/accent blue: `#8ec2f2`;
- CON/accent warm: `#f2a07e`;
- secondary blue: `#a9b8d8`;
- typeface: Georgia/serif; base ~10.5pt; line-height ~1.49.

Layout:

- A4 PDF;
- text padding around 18mm horizontally;
- page margin around 13mm top / 12mm bottom;
- `-webkit-print-color-adjust: exact` for dark-background PDFs.

## 14. Rendering and visual checks

Recommended render path:

```text
HTML → Chromium/Playwright print-to-PDF → pdftoppm screenshot → human visual check → pypdf page count
```

Before sending/publishing:

- inspect first page visually;
- verify disclaimer appears before verdict;
- verify page count;
- verify graph captions;
- verify no model/session/internal path leakage;
- verify generated PDF and HTML point to the same `metadata.json`.

## 15. License and DOI notes

Recommended default unless OD decides otherwise:

- public digest: readable/shareable with clear license statement;
- concept DOI: series-level DOI for archive anchor;
- individual release deposits only when evidence bundle deserves a citable archive;
- DOI never implies scientific validation.

## 16. Retrospective releases #1/#2

If releases #1/#2 already exist publicly, add retroactive metadata/disclaimer rather than silently rewriting history.

Allowed retrospective metadata:

```json
{
  "order_swap_audit": {"status": "NOT_RUN", "waiver_reason": "retrospective_release_before_D1b"},
  "protocol_status": "PARTIAL"
}
```

Do not pretend old releases satisfied new gates if they did not.

## 17. Bibliography / references

A duel built on or inspired by a concrete publication must cite it. This makes
the "Footnotes / references" section (§11, item 16) a hard requirement, not an
optional flourish, whenever the duel was seeded by real prior work.

**Canonical citation style: APA 7.** Every `ref` string is a strictly canonical
APA 7 reference and nothing else (no annotation, no "citation not captured"
placeholder). One style, applied to every release going forward. For arXiv
preprints the APA 7 form is:

```text
Author, A. A., Author, B. B., & Author, C. C. (Year). Title in sentence case (arXiv:NNNN.NNNNN). arXiv. https://arxiv.org/abs/NNNN.NNNNN
```

The reference list is ordered alphabetically by first author (APA 7), not by
citation order. A citation must be real and verifiable — recovered from the
seeding record (the Research weekly digest carries full author/title/id) and,
where reachable, confirmed against the source. Paraphrase or "not captured" is
not an acceptable reference.

Rules:

- If the sidecar sets `seed_publication: true`, it MUST carry a non-empty
  `references` array, and the rendered digest (RU and EN) MUST contain a visible
  references section.
- Each `references` entry is an object:

```json
{ "ref": "<canonical APA 7 citation>", "url": "<url or null>", "role": "seed", "note": "<why the source is relevant>" }
```

  `ref` is required, non-empty, and in canonical APA 7. `role` is `seed` for the
  publication the duel was built on, `supporting` for other cited work. `note` is
  optional and carries the justification (why the source is relevant / how the
  duel uses it) — it is kept OUT of the canonical `ref` string and rendered as a
  separate annotation, so the citation itself stays strictly canonical.
- The rendered references section MUST carry the anchor class `refs` (e.g.
  `<div class="appendix refs">`) so the gate can confirm a declared bibliography
  is actually visible in the artifact.
- A duel that is not seeded by any concrete publication omits both fields; no
  bibliography is forced.

The gate blocks release when `seed_publication` is set without `references`, when
a `references` array is malformed, or when references are declared but the
rendered HTML has no `refs` section. References flow through to the public
`metadata.json`.
