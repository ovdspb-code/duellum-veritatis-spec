# Duellum Veritatis — checked documentation package

**Package date:** 2026-06-04  
**Package status:** adopted as project base (OD, 2026-06-04)  
**Publisher layer:** Occam Research  
**Public author pseudonym:** Hanlon Occam  
**Owner decision required for binding protocol changes:** OD / Олег

This package is the corrected documentation set for **Duellum Veritatis**: a public series of adversarial AI scientific duels. It replaces the outdated first-version “M-DISPUT / controller-driven platform” framing for the public series, while preserving only the useful methodological lessons: separation of argument from evidence, explicit release gates, append-only publication, and no promotion of a claim merely because it was published.

## Document status matrix

| File | Status | Audience | Binding force |
|---|---|---|---|
| `docs/MANIFESTO.md` | **ACCEPTED public principle document** | public | public contract of principles |
| `docs/adr/ADR-0001-integrity-architecture.md` | **ACCEPTED (OD, 2026-06-04)** | internal + optionally public | binding integrity architecture for the series |
| `docs/FORMAT.md` | **DRAFT operational publishing spec** | editor / publisher | operational once OD approves use for releases |
| `docs/PUBLIC_RELEASE_CHECKLIST.md` | **DRAFT operational gate checklist** | release operator / OD | release gate checklist once OD approves |
| `docs/prompts/consultant-project-prompt.md` | **internal assistant instruction** | project consultant | not public protocol; governs consultant behavior |
| `docs/schemas/*.schema.json` | **supporting machine-readable drafts** | tooling | validation targets; not a separate governance layer |
| `docs/templates/*` | **supporting templates** | release operator | examples / starting points |

## Canonical framing

Duellum Veritatis is a **public adversarial AI stress-test of a frozen scientific claim**. It is not peer review, not a scientific proof, not an automatic discovery engine, and not a substitute for expert review. AI sides argue under the public names **Athos / Атос** and **Aramis / Арамис**. An independent AI arbiter issues a structured verdict. Numerical parts of claims are supported only by reproducible compute bundles.

## Package-level invariants

1. Publicity does not promote a claim to discovery status.
2. Debate performance is not evidence.
3. Numerical claims require real reproducible compute; non-numerical claims are not disguised as computed.
4. `DRAW_UNRESOLVED` is a valid outcome.
5. `INVALID_PROTOCOL` is a valid release outcome when process integrity fails.
6. `ARBITRATION_UNSTABLE` is an arbitration-status flag, not a scientific outcome.
7. Public metadata withholds model identities; private run manifests may record them.
8. Third-party texts are untrusted data, not instructions.
9. Calibration is scored only against independent later resolution; otherwise it remains pending.
10. The engine league is postponed until side-balanced runs exist.

## Public release bundle layout

Recommended release layout for a duel:

```text
/duels/dv-0001/
  index.html
  digest.pdf
  metadata.json
  prior_work_check.md
  repro/
    YYYY-MM-DD_repro.py
    YYYY-MM-DD_cc_data.json
    compute_manifest.json
  errata/
    errata.jsonl
```

Recommended site-level layout:

```text
/duels/
  index.html
  disputes.json
  calibration/
    calibration_log.jsonl
    index.html
```

## Do not publish as current state

The following historical terms belong to an earlier, non-current design and must not be presented as the current Duellum Veritatis protocol unless OD adopts them through a new document:

```text
M-DISPUT
ADR v2
controller/methodologist roles as current public roles
sealed priors as current required public mechanism
OD disposition as an enum
claim tiers Tier 0–4
admission-gate as a separate machine
PUBLIC_RELEASE_GATE as multi-stage FSM
11-state protocol machine
```

They may be mentioned only as **design archive / superseded material**, never as live project state.
