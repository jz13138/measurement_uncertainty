# GUM uncertainty evaluation of a frequency-dependent measurement series

Evaluation of a 15-times repeated measurement sweep from **10 kHz to 200 MHz** according
to *JCGM 100:2008* — *Evaluation of measurement data — Guide to the expression of
uncertainty in measurement* (GUM).

The measured quantity is a **level in dBm**, reported as

$$\hat\mu = \bar x \pm U(k{=}2), \qquad U = 2\,u_c = 2\sqrt{u_A^2 + u_B^2}$$

for every one of the 37 frequency points (555 individual readings).

## Contents

| File | Description |
|---|---|
| `GUM_Messwertanalyse.ipynb` | The complete analysis — run this first, it regenerates everything below |
| `GUM_whisker_boxplot_k2.png` | **Whisker box plot for the coverage factor `k = 2`**: box = `x̄ ± u_A`, whiskers = `x̄ ± U` |
| `Tukey_with_GUM_interval.png` | Tukey box plot of the 15 raw readings with the GUM interval overlaid |
| `Response_with_coverage_band.png` | Frequency response with the coverage band, and `s` / `u_A` / `U` vs. frequency |
| `Run_trend_diagnostic.png` | Readings vs. run index for the eight steepest groups |
| `Relative_expanded_uncertainty.png` | Relative expanded uncertainty `U/x̄` over the sweep |
| `GUM_uncertainty_budget.csv` | Full budget per frequency point (`u_A`, `u_B`, `u_c`, `ν_eff`, `U`, coverage, trend flag) |
| `TypeA_statistics.csv` | `n`, mean, `s`, `u_A`, min, max per frequency point |
| `Statistical_tests.csv` | Shapiro–Wilk, Grubbs, linear run trend, split-half test per group |
| `Run_trend_analysis.csv` | Trend-aware Type A evaluation (GUM Annex H) |

## Requirements

```bash
pip install numpy pandas scipy matplotlib
```

Python 3.9+ (developed and verified with 3.12, numpy 1.26, pandas 3.0, scipy 1.11,
matplotlib 3.6). Run the notebook top to bottom; every figure and CSV is written next to it.

## Method summary

1. **Type A** (GUM 5.4): `u_A = s/√n` with `n = 15`, so `ν_A = 14`.
2. **Type B**: resolution of the recording, `Δ/√12`, rectangular distribution.
   The remaining components (calibration of the reference, frequency axis, probe matching,
   drift, display reading) are configurable in `OTHER_TYPE_B` at the top of the notebook
   and enter the budget automatically.
3. **Combination** (GUM 5.1) with **Welch–Satterthwaite** effective degrees of freedom,
   then **expansion** with `k = 2` (GUM 6.2.2). The actual coverage probability is
   reported per frequency point, because `k = 2` only approaches 95 % in the limit
   `ν → ∞` — with `ν_eff ≈ 14` it is ≈ 93.5 %. For exactly 95 % use `k = 2.144`.
4. **Rounding** per GUM 8.1.3 / 5.3.2: two significant digits, three if the leading digit
   is 1 or 2 (`gum_round()`).

### Pre-tests the GUM requires

Normality (Shapiro–Wilk), outliers (Grubbs, flag-only by default) and independence of the
readings (linear run trend plus a split-half Welch *t*-test). Where a run trend is
detected, the Type A term is taken about the fitted trend instead of about the mean
(GUM 4.1.2, Annex H); both the trend-aware and the plain `s/√n` budget are printed so the
effect is visible.

## Key results

* Repeatability spans a factor of 30 across the sweep: `s = 0.030 … 0.94` dBm,
  `U(k=2) = 0.017 … 0.22` dBm. 15 repetitions buy a factor of `√15 = 3.87`.
* Relative expanded uncertainty: median `0.39 %`, worst case `6.7 %` at `0.1 MHz`,
  best case `0.10 %` at `1.8 MHz`.
* **The measurand was not stationary in 19 of the 37 series** — most strongly at
  `0.1 (B)`, `0.135` and `0.182 MHz`, where the level rises monotonically across the 15
  runs. The equivalent Type B drift term reaches `1.52` dBm there, six times the random
  Type A term. A single mean value is therefore only conditionally valid for these groups.
* The duplicated `0.100 MHz` entry is **not** a repetition (Welch `t = 98`,
  `p ≈ 1e-26`) but two different measurement conditions, so it is evaluated as the
  separate groups `0.1 (A)` and `0.1 (B)`.

## Caveat

The committed budget contains only the Type B component that can be justified from the
raw data file alone (resolution). A complete GUM budget for a report additionally needs
the calibration uncertainty of the reference and the uncertainty of the frequency axis —
the sweep points are only quoted to two or three digits. This is listed as an open item
in the notebook.