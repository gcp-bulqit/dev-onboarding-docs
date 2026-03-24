# `packages/vendors-bulqit-score`

## Paths

| Item | Path |
|------|------|
| Math | `packages/vendors-bulqit-score/src/calculations.ts` |

## Exported functions (synopsis)

- **`calculateBulqitScore`** — Composite score from responsiveness, commitment, reviews, etc.
- **`calculateBayesianAverageScore`**, **`calculateResponsivenessScore`**, **`calculateCommitmentScore`** — Sub-scores.
- **`parseNumber`**, **`DEFAULT_BULQIT_SCORE`**, **`DEFAULT_BULQIT_WARNING`** — Helpers/constants.

## Behavior synopsis

**Deterministic, pure functions**—no DB inside the calculators. Callers pass **`VendorQualityAndCommitmentStats`**-like structs; results feed vendor rankings and admin views. Temporal’s **`update-vendor-scores`** workflow can batch recompute.

**Features controlled:** Marketplace **trust signals** and vendor ranking inputs.
