# GRIN2A Mutant Electrophysiology Analysis

**Bootstrap and empirical Bayes ridge regression analysis of NMDA/AMPA receptor current kinetics across GRIN2A missense mutations**

*Voluntary research collaboration, Dr Andrew Penn's lab · University of Sussex*

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/acpennlab/statistics-resampling-online/jammy-docker?urlpath=git-pull%3Frepo%3Dhttps%253A%252F%252Fgithub.com%252Frashikafouzia9-png%252FGRIN2A-multivariate%26urlpath%3Dlab%252Ftree%252FGRIN2A-multivariate%252F%26branch%3Dmaster)
[![Language](https://img.shields.io/badge/language-Octave-blue.svg)](https://octave.org/)

---

## The question

GRIN2A encodes the GluN2A subunit of the NMDA receptor. Missense mutations in this subunit are linked to epilepsy, but different mutations can have very different functional consequences on receptor behaviour — some slow the receptor's kinetics, some speed them up, some barely change them at all.

This project asks: **do individual GRIN2A missense mutations (K669N, L812M, C436R, T531M, R518H) measurably alter NMDA and AMPA receptor current kinetics relative to wildtype, and which kinetic properties are actually affected?**

## Data

Paired transfection electrophysiology recordings (`data/n2a_mutant.xlsx`): for each of 6 animals, cells were recorded under a mutant (+) and control (–) transfection condition. Eleven kinetic outcome measures were extracted per recording pair, covering both NMDA and AMPA components:

`peakNMDA, decayNMDA, chargeNMDA, dt50NMDA, fwhmNMDA, peakAMPA, decayAMPA, chargeAMPA, riseAMPA, dt50AMPA, fwhmAMPA`

Outcomes were log transformed and expressed as within pair differences (mutant − control), giving 125 paired observations across 6 mutation groups (5 mutants + WT control comparisons).

## Why this needed more than a standard linear model

With only 6 animals contributing repeated, non-independent recordings, a plain OLS model would understate the true uncertainty — recordings from the same animal aren't independent samples, and a standard model would treat them as if they were. Two things had to be handled together:

**1. Clustering.** For each of the 11 outcomes, a design effect (Deff) was estimated by comparing Bayesian bootstrap models with and without animal level clustering (`bootlm` with `clustid`). Deff ranged from ~1.5 to ~23 across outcomes — meaning some kinetic measures showed almost no animal level dependency, while others (like `fwhmNMDA` at Deff ≈ 23) were heavily clustered by animal. Ignoring this would have made several effects look far more certain than they actually are.

**2. Small sample, multi outcome inference.** With 11 correlated outcomes and only 5-6 mutation groups, an empirical Bayes ridge regression (`bootridge`, .632 bootstrap tuned, 1999 resamples) was used instead of per outcome OLS. This shrinks noisy per outcome estimates toward a common structure, uses the animal level Deff to correctly inflate residual variance and degrees of freedom, and returns Bayes factors and bootstrap stability scores (the proportion of bootstrap resamples in which a coefficient's sign/significance held) as evidence measures rather than relying on p-values alone in an underpowered design.

## What the results show

Comparing WT vs. all mutants pooled (Contrast A), three NMDA kinetic measures stood out with strong, stable evidence:

| Outcome | Coefficient | Bayes Factor (BF10) | Bootstrap stability |
|---|---|---|---|
| decayNMDA | +0.396 | 52,388 | 100% |
| dt50NMDA | +0.397 | 1,315,889 | 100% |
| fwhmNMDA | +0.377 | 9,463,895 | 100% |

These three are consistently and strongly evidenced across the bootstrap resamples — GRIN2A mutations as a group slow NMDA current decay kinetics relative to WT. AMPA associated outcomes (peak, decay, charge, rise, dt50, fwhm) showed weak or inconclusive evidence (BF10 close to 1, stability well under the 97.5% threshold used here), suggesting the functional effect of these mutations is concentrated in NMDA current decay dynamics rather than amplitude or AMPA co-transmission.

Individual mutation vs mutation contrasts (e.g. loss of function vs. gain of function groupings) are computed in the notebook but are noisier, as expected with 17-29 paired observations per mutation.

*Full output, contrast definitions, and per-mutation estimated marginal means are in [`GRIN2A-mutant.ipynb`](GRIN2A-mutant.ipynb), with figures saved to `output/`.*

## Method summary

```
Paired (+/-) log current-kinetic differences
        │
        ▼
Per outcome bootstrap linear models (bootlm, Wild bootstrap-t)
for initial contrasts and sanity checks
        │
        ▼
Animal level design effect estimation
(Bayesian bootstrap, clustered vs. unclustered)
        │
        ▼
Empirical Bayes ridge regression (bootridge, .632 bootstrap-tuned,
Deff adjusted residual variance and degrees of freedom)
        │
        ▼
Bayes factors + bootstrap stability scores per outcome × contrast
```

## Running it

Click the Binder badge above to launch the notebook directly — no local setup needed. To run locally, you need GNU Octave with the `statistics resampling` package:

```octave
pkg install -forge statistics-resampling
pkg load statistics-resampling
```

Then open `GRIN2A-mutant.ipynb` in JupyterLab.

## Status

This is an active collaboration with Dr Andrew Penn — the interaction model (mutation × transfection) and full contrast set are still being extended. This README reflects the analysis as of 2nd August 2026.
