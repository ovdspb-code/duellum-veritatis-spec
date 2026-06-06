# PUBLIC_RELEASE_CHECKLIST — Duellum Veritatis

**Статус:** DRAFT operational release gate  
**Дата:** 2026-06-05  
**Scope:** lightweight manual gate before `approved` sidecar  
**Non-goal:** this is not a state machine, not a controller platform, not a multi-stage enterprise gate

This checklist is passed before any public HTML/PDF/site/Zenodo release. It operationalizes ADR-0001 D1–D10 plus the D1b order-swap artifact and the ADR-0002 D11–D13 evidence/debate-separation artifacts, mirroring what `duellum_gate` enforces.

## 0. Release identity

- [ ] `duel_id` exists and is unique.
- [ ] Release path exists: `/duels/<duel_id>/`.
- [ ] `metadata.json` exists and validates against `docs/schemas/metadata.schema.json`.
- [ ] Public `series` value is `Duellum Veritatis`.
- [ ] Public `status` value is `public_adversarial_ai_stress_test`.

## 1. Claim freeze

- [ ] Frozen claim text is present.
- [ ] `frozen_at` timestamp is present.
- [ ] `claim_type` is one of: `numeric`, `conceptual`, `literature_only`, `mixed`.
- [ ] Digest uses the same frozen claim text as `metadata.json`.
- [ ] No later section silently strengthens the claim.

## 2. Side assignment / engine withholding

- [ ] PRO/CON assignment was fixed before the duel.
- [ ] `side_assignment_seed_hash` is recorded.
- [ ] Public side names are only `Athos/Атос` and `Aramis/Арамис` or neutral `PRO/CON`.
- [ ] Public metadata sets `model_id_public = "withheld"`.
- [ ] No concrete model name, engine version, session ID, sandbox path or relay name appears in public HTML/PDF/metadata.

## 3. Arbiter integrity

- [ ] Arbiter is outside the duel roster.
- [ ] Arbiter input was identity-normalized.
- [ ] Arbiter input contains no model names/session metadata.
- [ ] `order_swap_audit.status` is recorded as `PASS`, `UNSTABLE`, `NOT_RUN`, or `WAIVED`.
- [ ] For a new release after ADR-0001 ACCEPTED: `order_swap_audit.status = PASS`, unless OD waiver is recorded.
- [ ] If `order_swap_audit.status = UNSTABLE`, then `outcome.enum = DRAW_UNRESOLVED` and `arbitration_status = ARBITRATION_UNSTABLE`.
- [ ] If `order_swap_audit.status = NOT_RUN` or `WAIVED`, `waiver_reason` is present (and, for a non-retrospective release, `waiver_approved_by = OD`).
- [ ] **D1b artifact:** if `order_swap_audit.status` is `PASS` or `UNSTABLE`, `order_swap_audit_file` points to an artifact that validates against `docs/schemas/order_swap_audit.schema.json` (both input + both output hashes), and its `stability` matches `order_swap_audit.status`. The auto-pipeline (`duel_packet.py`) materializes this file from the relay capture — confirm it exists rather than hand-writing hashes.
- [ ] Verbosity ratio was checked or explicitly marked `not_measured` with reason.

## 4. Outcome consistency

- [ ] `outcome.enum` is one of: `PRO`, `CON`, `DRAW_UNRESOLVED`, `INVALID_PROTOCOL`.
- [ ] `outcome.basis` is one of: `real_compute`, `literature_review`, `conceptual_analysis`, `mixed`, `protocol_failure`.
- [ ] `arbitration_status` is one of: `STABLE`, `ARBITRATION_UNSTABLE`, `NOT_RUN`, `WAIVED`.
- [ ] `protocol_status` is one of: `PASS`, `PARTIAL`, `FAIL`.
- [ ] `ARBITRATION_UNSTABLE` is not used as an outcome.
- [ ] If `protocol_status = FAIL`, then `outcome.enum = INVALID_PROTOCOL` unless OD records an explicit reason.
- [ ] Digest renders `DRAW_UNRESOLVED` as a valid outcome, not as a pipeline failure.

## 5. Compute / reproducibility

For `claim_type = numeric` or `basis = real_compute|mixed`:

- [ ] Repro script exists.
- [ ] Input data or data manifest exists.
- [ ] `repro/compute_manifest.json` exists.
- [ ] `compute_manifest.json` validates against `docs/schemas/compute_manifest.schema.json`.
- [ ] Script hash is present.
- [ ] Input hashes are present.
- [ ] Output hashes are present.
- [ ] Environment is recorded.
- [ ] Seed or deterministic/no-seed note is recorded.
- [ ] Metric value and tolerance are recorded for outcome-deciding metrics.
- [ ] Rerun instruction is present.
- [ ] Digest numbers and graphs can be traced to outputs listed in `compute_manifest.json`.

For non-numerical releases:

- [ ] Digest does not describe the verdict as computed.
- [ ] Numerical illustrations, if any, are explicitly marked as non-decisive unless in compute manifest.

## 5b. Evidence / debate separation (ADR-0002 — enforced by the executable gate)

These artifacts are required by `duellum_gate` for every public duel (ADR-0002
ACCEPTED 2026-06-06); the gate enforces them, so a release that skips them is
blocked. The point is the No.3 failure: debate, compute and accepted evidence
blurred together and the digest claimed more than the test showed.

- [ ] `epistemic_status` is one of: `scout`, `protocol_pilot`, `methodological_case_study`, `scientific_result`.
- [ ] `claim_evidence_matrix` is referenced and validates against `docs/schemas/claim_evidence_matrix.schema.json` (each sub-claim mapped to its evidence).
- [ ] `arbiter_report` is referenced and validates against `docs/schemas/arbiter_report.schema.json`, and its `outcome_classification` equals the sidecar `outcome`.
- [ ] For `claim_type = numeric` or `basis = real_compute|mixed`: `evidence_task` is referenced, validates against `docs/schemas/evidence_task.schema.json`, and its decisive `observable` carries `evidence_basis` (one of `simulation`, `semi_analytic`, `analytic`, `empirical_data`).
- [ ] `scientific_result` is claimed ONLY when the release is numeric/real_compute/mixed AND the decisive observable is `paper_faithful` AND its `evidence_basis` is above bare `simulation` (`semi_analytic`/`analytic`/`empirical_data`). A toy/proxy observable, or a paper-faithful observable measured by finite-size simulation alone, may publish as `scout`/`protocol_pilot`/`methodological_case_study` but **not** as `scientific_result`.

## 6. Prior-work / novelty

- [ ] `prior_work_check.md` exists.
- [ ] Search date is recorded.
- [ ] Search queries are recorded.
- [ ] Closest prior works are listed or absence is justified.
- [ ] `novelty_status` is one of: `not_an_originality_claim`, `likely_known`, `unclear`, `candidate_new_boundary`.
- [ ] Digest does not use “new result”, “discovery”, “first shown” unless prior-work check supports it.
- [ ] Default wording says the release is a stress-test, not an originality claim.

## 7. Third-party text handling

If target is a third-party paper/text:

- [ ] Target text is treated as untrusted data.
- [ ] Visible extraction was separated from hidden/suspicious instruction screening.
- [ ] Extracted representation hash is recorded.
- [ ] Only quoted, delimited excerpts were passed into debate bundles.
- [ ] No target-text instruction can override system/role/protocol prompts.
- [ ] If hidden prompts or suspicious content were detected, contamination note or quarantine is recorded.
- [ ] Digest critiques the claim/evidence, not author identity.

If no third-party text was used:

- [ ] `third_party_text_handling = not_applicable`.

## 8. Public wording

- [ ] Mandatory RU/EN status disclaimer appears before verdict.
- [ ] Forbidden wording matrix was checked.
- [ ] “AI proved…” wording is absent.
- [ ] “Peer review replacement” wording is absent.
- [ ] “Publication = validation/discovery” wording is absent.
- [ ] Engine league/ranking wording is absent unless explicitly a future/postponed note.
- [ ] What the result supports is explicit.
- [ ] What the result does **not** support is explicit.

## 9. Append-only / errata

- [ ] `disputes.json` append-only diff was reviewed.
- [ ] No existing public release was silently overwritten.
- [ ] If correcting a prior release, errata entry exists.
- [ ] If redaction/quarantine occurred, public note exists.
- [ ] Release package manifest lists hashes of public files.

## 10. Publication gate

- [ ] `approved` sidecar exists.
- [ ] `approved_sidecar_hash` is recorded in `metadata.json`.
- [ ] OD OK for public site/Zenodo step is recorded if required.
- [ ] `publication_gate.status = PASSED`, or release is blocked.
- [ ] Known enforcement gaps are recorded.

## 11. Final preflight statement

Before publication, the release operator writes one concise statement:

```text
I checked frozen claim, identity withholding, arbitration status, outcome enum, compute/prior-work artifacts, third-party text handling, forbidden wording, append-only diff, and approved sidecar. Publicity does not promote the claim.
```

If any line above cannot be checked, do not publish unless OD records a specific waiver.
