# Clearance → Coverage: definitions

*Version 0.1 — AI in Health, Clinical and Market Access Evidence Monitor*

"Coverage" is not one thing. A provisional DiGA listing, a three-year NTAP, a Category III
CPT code that often pays nothing, and an LCD in a single MAC region are all called "reimbursed"
in press releases, and they mean entirely different things to a manufacturer.

This document fixes the definitions used in the tracker. It is published openly and may be
cited or adopted. If you disagree with a rule, say so — the rules improve by being argued
with, and a shared vocabulary is worth more than a private one.

---

## The clock

**T₀ (start) = first market authorisation in the jurisdiction's regulatory system.**

| Market | T₀ is | Source |
|---|---|---|
| US | FDA decision date (510(k), De Novo, or PMA) | openFDA |
| EU/DE/FR | CE mark issue date under MDR | Manufacturer disclosure; EUDAMED where available |
| UK | UKCA mark, or CE mark under current recognition | Manufacturer disclosure |

**T₁ (stop) = the date a payer first becomes obliged, or explicitly permitted, to pay.**
Not the date a code exists. Not the date a pilot starts. Payment must be *obtainable*.

**Days-to-coverage = T₁ − T₀**, per market, per device.

Where T₀ is unknown or undisclosed, the device is recorded but excluded from median
calculations. **An unknown date is never estimated.**

---

## What counts as coverage — by market

### United States

| Status | Counts as covered? | Why |
|---|---|---|
| **NTAP granted** | **Yes** | Payment is obtainable, though time-limited (typically 2–3 years). Recorded with its expiry. |
| **CPT Category I + assigned RVUs** | **Yes** | Routine payment pathway. |
| **CPT Category III** | **No** | Tracking code. Payment is discretionary and frequently zero. Recorded as `code_only`. |
| **NCD (positive)** | **Yes** | National obligation. |
| **LCD (positive)** | **Partial** | Recorded with the MAC region. Counts as covered *for that region only*; excluded from national medians. |
| **Commercial policy only** | **Partial** | Recorded with the payer named. Not counted in Medicare medians. |

### Germany

| Status | Counts as covered? | Why |
|---|---|---|
| **DiGA — permanent listing** | **Yes** | Evidence accepted; statutory reimbursement. |
| **DiGA — provisional listing** | **Yes, flagged** | Payment is real during the trial period, but conditional. Recorded as `provisional`; reported separately. |
| **DiGA — delisted** | **No** | Recorded as `failed`, with the date. **Failures are as informative as successes and are not deleted.** |
| **NUB Status 1** | **Yes** | Hospital may negotiate payment. |
| **NUB Status 2/3** | **No** | No additional-payment route. |
| **NUB Status 4** | **No** | No *additional* NUB payment — the method is already reflected within the DRG. |

### France

| Status | Counts as covered? | Why |
|---|---|---|
| **PECAN early access** | **Yes, flagged** | Payment obtainable ahead of full assessment. Recorded as `early_access`. |
| **LPPR listing** | **Yes** | Definitive reimbursement. |
| **HAS/CNEDiMTS favourable opinion, not yet listed** | **No** | Opinion is not payment. Recorded as `assessed_not_paid`. |

### United Kingdom

| Status | Counts as covered? | Why |
|---|---|---|
| **MedTech Funding Mandate** | **Yes** | Commissioners obliged to fund. |
| **NICE EVA — recommended for use with evidence generation** | **No** | Conditional use, no funding obligation. Recorded as `conditional_use`. |
| **NICE guidance (positive), no funding mandate** | **Partial** | Recorded; excluded from medians. |

---

## Rules that keep the series honest

1. **Failures stay in.** Delisting, withdrawal, expiry, and negative decisions are recorded
   with dates, alongside successes.
2. **No backfilled estimates.** If a date can't be sourced, the cell is `unknown`, not a guess.
3. **Every row carries its source.** URL or document reference, captured at logging time,
   because these pages get rewritten in place.
4. **Provisional ≠ permanent.** They are separate statuses and are never merged in a median.
5. **The tracker records what a payer *would* pay, not what anyone was actually paid.**
   Utilisation is a different question and is out of scope.

---

## Status vocabulary

`covered` · `covered_provisional` · `covered_early_access` · `covered_regional` ·
`code_only` · `assessed_not_paid` · `conditional_use` · `pending` · `refused` ·
`withdrawn` · `expired` · `unknown`

---

Corrections and disagreements are welcome — open an issue.

---

## Evidence that won coverage (the `evidence` block)

Time-to-coverage records *how long*. The `evidence` block records *what evidence
supported the decision*. Attach it to any **covered** decision. Aggregates (share by design, endpoint mix, RCT rate, accuracy-only
count) are published; the row-level detail stays private.

```yaml
coverage:
  us:
    - status: covered
      mechanism: NTAP
      date: 2025-10-01
      evidence:
        design: rct
        endpoint: clinical_outcome
        comparator: standard_of_care
        n: 1240
        source: https://doi.org/...
```

### `design` — the study behind the winning dossier

| Value | Meaning |
|---|---|
| `rct` | Randomised controlled trial. |
| `prospective_obs` | Prospective observational / registry / real-world cohort. |
| `retrospective` | Retrospective analysis of existing data. |
| `modelling` | Economic model or simulation only; no primary clinical study. |
| `none` | Cleared/covered without a study of the device itself (e.g. predicate reliance). |
| `unknown` | Not disclosed. Recorded, never guessed. |

### `endpoint` — the argument that convinced the payer

This field records whether the decision rested on a diagnostic-accuracy measure or on a
downstream clinical, economic, or workflow outcome.

| Value | Meaning |
|---|---|
| `clinical_outcome` | A patient outcome changed (mortality, morbidity, detection→treatment). |
| `diagnostic_accuracy` | Sensitivity / specificity / AUC only — no downstream outcome shown. |
| `economic` | Cost, cost-effectiveness, utilisation, or length-of-stay. |
| `workflow` | Time saved, throughput, or read-burden reduction. |
| `composite` | A bundle spanning more than one of the above. |
| `unknown` | Not disclosed. |

### `comparator` — what it was measured against

| Value | Meaning |
|---|---|
| `standard_of_care` | Compared to current clinical practice. |
| `no_ai` | Clinician with vs without the AI. |
| `placebo` | Sham / placebo control (rare, mostly digital therapeutics). |
| `none` | Single-arm; no comparator. |
| `unknown` | Not disclosed. |

### Rules

1. **Only on `covered*` decisions.** Evidence that won a *provisional* or *early-access*
   listing counts, and is flagged by that status — a lower bar than permanent listing.
2. **`n` and `source` are optional but strongly encouraged** — the citation is what makes
   a row defensible and reusable.
3. **`unknown` over a guess, always.** A fabricated endpoint quietly corrupts the one
   statistic that makes this dataset worth citing.
4. The public site shows only shares and counts. No device, date, `n`, or `source` is
   ever rendered.
