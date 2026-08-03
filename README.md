# GRIN2A Mutant Electrophysiology Analysis

Bootstrap and empirical-Bayes ridge regression on NMDA/AMPA current kinetics across GRIN2A missense mutations.

*Voluntary research collaboration with Dr Andrew Penn, University of Sussex.*

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/acpennlab/statistics-resampling-online/jammy-docker?urlpath=git-pull%3Frepo%3Dhttps%253A%252F%252Fgithub.com%252Frashikafouzia9-png%252FGRIN2A-multivariate%26urlpath%3Dlab%252Ftree%252FGRIN2A-multivariate%252F%26branch%3Dmaster)

## What this is

GRIN2A codes for the GluN2A subunit of the NMDA receptor, and different missense mutations in it show up in epilepsy — but they don't all do the same thing functionally. Some slow the receptor down, some barely touch it. I'm looking at whether five specific mutations (K669N, L812M, C436R, T531M, R518H) actually change NMDA/AMPA current kinetics relative to wildtype, and if so, which properties are affected.

## Data

Paired transfection recordings — each of 6 animals contributed a mutant (+) and control (–) recording. 11 kinetic measures per pair, covering both NMDA and AMPA components (peak, decay, charge, rise time, half decay time, FWHM). Outcomes are log transformed pair differences, 125 observations total across the 5 mutant groups.

## Why not just run a linear model per outcome

Because 6 animals contributing repeated recordings isn't 6 independent samples, and pretending it is would understate the real uncertainty. Two things had to be dealt with:

**Clustering.** For each of the 11 outcomes I estimated a design effect by comparing clustered vs. unclustered Bayesian bootstrap models. It ranged from about 1.5 to 23 depending on the outcome — `fwhmNMDA` at Deff ≈ 23 means recordings from the same animal are very much not independent for that measure, while some other outcomes barely showed clustering at all. Ignoring this would make several effects look far more certain than they are.

**Too many correlated outcomes, too few groups.** With 11 outcomes and 5-6 groups, per-outcome OLS is asking for trouble. I used empirical Bayes ridge regression instead (`bootridge`, .632 bootstrap, 1999 resamples), which shrinks noisy estimates and folds the animal level design effect into the residual variance and degrees of freedom, then reports Bayes factors and bootstrap stability (how often a coefficient held sign/significance across resamples) instead of leaning on p values in an underpowered design.

## Results

Pooling all mutants vs. WT (Contrast A), three NMDA measures came out strong:

| Outcome | Coefficient | BF10 | Stability |
|---|---|---|---|
| decayNMDA | +0.396 | 52,388 | 100% |
| dt50NMDA | +0.397 | 1,315,889 | 100% |
| fwhmNMDA | +0.377 | 9,463,895 | 100% |

Those BF10 values are enormous, and consistent across every bootstrap resample — the mutations as a group slow NMDA decay kinetics. AMPA-side outcomes didn't show anything close to this (BF10 near 1, stability well under threshold), so whatever these mutations are doing functionally, it's concentrated in NMDA decay, not amplitude or AMPA co-transmission.

Individual mutation-vs-mutation contrasts are in the notebook too but noisier, as you'd expect with 17-29 pairs per mutation rather than the pooled 125.

Full contrasts, marginal means, and figures: [`GRIN2A-mutant.ipynb`](GRIN2A-mutant.ipynb), plots in `output/`.

## Running it

Binder badge above launches it with no setup. Locally you need Octave + `statistics-resampling`:

```octave
pkg install -forge statistics-resampling
pkg load statistics-resampling
```

## Status

Active — this is still ongoing work with Dr Penn, the contrast set and interaction model are being extended. Last updated [August 2026].

## References


