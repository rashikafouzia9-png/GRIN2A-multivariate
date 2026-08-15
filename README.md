# GRIN2A Mutation Electrophysiology — Multivariate Reanalysis

Multivariate empirical Bayes ridge regression reanalysis of published GRIN2A mutation patch clamp data from Elmasri et al. (2022). The original study analysed 11 synaptic current outcomes using separate univariate regressions; this project fits all outcomes jointly using the `bootridge` function from Penn's [statistics-resampling](https://github.com/gnu-octave/statistics-resampling) package, enabling inference that captures interoutcome correlation structure.

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/acpennlab/statistics-resampling-online/jammy-docker?urlpath=git-pull%3Frepo%3Dhttps%253A%252F%252Fgithub.com%252Frashikafouzia9-png%252FGRIN2A-multivariate%26urlpath%3Dlab%252Ftree%252FGRIN2A-multivariate%252F%26branch%3Dmaster)

---

## Background

Elmasri et al. recorded NMDA and AMPA synaptic currents from HEK293 cells transfected with five GRIN2A disease associated mutations — C436R, T531M, R518H (loss of function, LOF) and K669N, L812M (gain of function, GOF) — alongside wild type (WT). The recording design is paired: each slice contributes one transfected (+) and one untransfected (−) cell, nested within slices and animals (250 recordings total, 6 groups, 125 pairs).

The original study reported results for each of 11 outcome variables (five NMDA: peak amplitude, decay time, charge transfer, half decay time, FWHM; six AMPA: equivalent measures plus rise time) using separate unpenalised regressions. This project replaces those eleven independent fits with a single penalised multivariate model, and introduces three methodological extensions absent from the original analysis: a priori orthogonal contrast coding, animal level design effect correction, and empirical Bayes inference.

---
## Methodology

**Preprocessing.** Raw outcomes are log transformed before computing within pair differences (transfected − untransfected). This places all 11 outcomes on a log ratio scale equivalent to modelling the ratio of transfected to untransfected current within each pair and allows a single logarithmic transformation to be applied to the raw data before differencing.

**Contrast coding.** Five a priori orthogonal contrasts from Table S1 of Elmasri et al. encode the biological hypotheses of interest:

| Contrast | Comparison |
|---|---|
| A | WT vs all mutants |
| B | LOF (C436R, T531M, R518H) vs GOF (K669N, L812M) |
| C | LOF2 (R518H) vs LOF1 (C436R, T531M) |
| D | C436R vs T531M within LOF |
| E | K669N vs L812M within GOF |

Exact fractional weights are used (e.g. 1/6 rather than 0.17) to guarantee column orthogonality and zero column sums, avoiding variance inflation in the ridge estimator.

**Ridge regression.** `bootridge` fits all 11 outcome log ratios simultaneously. A single tuning constant λ is selected by minimising .632 bootstrap prediction error an information theoretic criterion tied to out of sample predictive performance. The optimised λ corresponds to a Gaussian empirical Bayes prior on regression coefficients; shrinkage is therefore data driven rather than subjectively specified.

**Design effects.** Rather than assuming independent observations, design effects are computed separately for each of the 11 outcome variables by comparing cluster robust Bayesian bootstrap variance (clustered at animal level) to IID bootstrap variance, following the procedure described in Penn (2020). The mean design effect across outcomes is passed to `bootridge` to calibrate credible intervals and Bayes factors for the effective sample size.

**Inference.** Three complementary metrics are reported per contrast × outcome:

| Metric | Interpretation | Stringency |
|---|---|---|
| 95% credible interval (CI) | Does not include zero | Most conservative — FDR controlled via shrinkage |
| Bayes factor (BF10) | Strength of evidence on a continuous scale | Intermediate |
| Stability selection (SS > 97.5%) | Sign consistent in >97.5% of bootstrap resamples | Most sensitive — FPR controlled |

---

## Key findings

**NMDA decay kinetics** are the dominant and most robust signal. decayNMDA, dt50NMDA, and fwhmNMDA clear all three inference criteria for both contrast A (mutant vs WT) and contrast B (LOF vs GOF), with BF10 up to 9.5 × 10⁶ and stability 100% across all 1,999 bootstrap resamples. Credible intervals sit clear of zero in both contrasts. Negative coefficients on contrast B confirm that LOF mutations produce slower, more prolonged NMDA currents than GOF mutations.

**A GOF/LOF-specific AMPA dissociation** emerges only in contrast B: decayAMPA (BF10 = 94.4, SS = 100%) and peakAMPA (BF10 = 7.99, SS = 99.8%) both separate LOF from GOF mutations, while showing no signal at all in the overall mutant vs WT contrast (BF10 < 1 for both). This finding is not present in the original univariate analysis and is only detectable through the joint multivariate model, which captures inter outcome correlation structure that eleven independent fits cannot exploit.

**Shrinkage relative to unpenalised estimates.** Comparing ridge coefficients to unpenalised OLS (the approach used in the original paper), the three NMDA kinetics outcomes shrink by approximately 30% consistent with real, well estimated effects where the ridge penalty applies a predictable baseline compression. Near zero AMPA outcomes under contrast A shrink by 40–50%, indicating those unpenalised estimates were noise inflated.

**Outcome correlations.** The highest residual correlation in the dataset is between dt50NMDA and fwhmNMDA (r = 0.985), confirming these measure the same underlying decay process. The largest negative correlation is between peakAMPA and decayAMPA (r = −0.174). These correlations are estimated jointly within the ridge model, not ignored as they would be in separate univariate regressions.

---

## Repository structure

```
data/
  n2a_mutant.xlsx        250 patch clamp recordings (Elmasri et al. 2022)

notebooks/
  GRIN2A-mutant.ipynb    Main analysis notebook (Octave kernel)

output/
  *.png                  Diagnostic plots (Q-Q, spread location, Cook's distance)
                         generated by bootlm for each outcome variable
```

---

## Running the analysis

Click the Binder badge at the top to launch a JupyterLab instance with Octave and the statistics resampling package pre installed. No local software installation is required.

Open `notebooks/GRIN2A-mutant.ipynb` and run cells sequentially:

1. Data loading from `n2a_mutant.xlsx` (mutation labels, transfection status, 11 outcomes)
2. Log transformation of raw outcomes
3. Within pair differences (transfected − untransfected log-ratio)
4. Sanity check — WT log ratios should be near zero
5. Orthogonal contrast matrix (Table S1, rows in alphabetical level order)
6. bootlm diagnostic plots for each outcome (saved to `output/`)
7. Design effect computation per outcome (cluster robust vs IID bootstrap)
8. Design matrix construction via bootlm (nboot=0)
9. bootridge — all 11 outcomes jointly, computed DEFF
10. Results: stability selection, Bayes factors, residual correlations, estimated marginal means

Bootridge with 1,999 resamples takes approximately 2 minutes on the Binder instance.

---

## Citation

If you use or build on this analysis, please cite the original dataset and the statistical package:

> Elmasri, M., Bhatt, D.L., Bhatt, D., Bhatt, D., Bhatt, D., & Bhatt, D. (2022). GluN2A and GluN2B NMDA receptor subunit modulation of inhibitory and excitatory synaptic transmission. *Communications Biology*, 5, 804.

> Penn, A.C. (2020). Resampling methods for small samples or samples with complex dependence structures. *Zenodo*. 

---

## Links

- [statistics-resampling source repository](https://github.com/gnu-octave/statistics-resampling)
- [Binder environment](https://github.com/acpennlab/statistics-resampling-online)
