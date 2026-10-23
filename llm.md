# ARTU: documentation and execution status

## Authority and scope

This describes the repository at source commit `11aefa42049c737c1e6d845a5428d29aa48dd506`. The repository provides a technical note and `K_alpha.zip`, containing a MATLAB coefficient lookup function and four tables. It is not a complete executable forecasting pipeline. This documentation does not alter the archived scientific source.

Scientific authority checked: [arXiv:2203.14959v1](https://arxiv.org/abs/2203.14959v1), Section 2.2.1, Eqs. 10–16 and the conclusions, plus the repository's `ARTU.pdf`, Algorithm 5. The publisher PDF has not been checked line by line. The purpose is a demanding, inexpensive benchmark against which extra model complexity must justify itself. ARTU is not universally the best predictor; the paper compares several references and their combination across settings and horizons.

## Execution status

**Corrected on 23 October 2026.** Until then the archived `K_alpha.m` declared input `Q` but tested `R`, undefined in its scope (`MATLAB:UndefinedFunction`). It now selects the table from `Q` (0, 1, 2, 3 for R = 0, 0.01, 0.05, 0.1; the values 0.01, 0.05 and 0.1 of R are also accepted), defaults to Q = 2 (R = 0.05, the paper's choice when the measurement error is unknown) and raises `K_alpha:Q` for any other value. The four tables are unchanged. Check against the paper: `K_alpha(0.4, 0.3, Q)` returns (K, α) = (−0.27, 0.60), (−0.25, 0.59) and (−0.23, 0.58) for R = 0.01, 0.05 and 0.1, the values of the paper's example (h = 1, ρ(1) = 0.4, ρ(2) = 0.3; MATLAB R2025b, 23 October 2026).

## Intended inputs and outputs

- `corrh`, `corr2h`: real, finite, dimensionless autocorrelations at forecast lags h and 2h of the same training clear-sky-index series. The source comment states -1 < correlation < 1; the stored interpolation grid spans -1:0.05:1. Grid coverage alone does not certify feasibility of an arbitrary pair of autocorrelations.
- `Q`: intended discrete selector 0, 1, 2, 3 for tables R0, R0.01, R0.05, R0.1. It is not a continuous percentage or an irradiance value. Do not silently choose a reliability level for the user: the function defaults to Q = 2 (R = 0.05), so state the value used.

In the paper, **R = variance(measurement noise) / variance(state)**. Thus R=0.05 is a variance ratio, not a claim of 5% irradiance-amplitude accuracy. Q is only an index into the available R tables. The paper describes a correlation grid step of 0.01; this archived interpolation function constructs a grid with step 0.05. These resolutions must not be conflated or advertised as an exact reproduction without checking the tables and coefficient-generation protocol.
- Outputs `K`, `alpha`: dimensionless coefficients interpolated in the tables. They match the paper's example; outside the grid (|ρ| > 1) `interp2` returns NaN. The current function implements no explicit validation of input bounds or missing data.

The four `.mat` files must be resolvable by MATLAB when the lookup is called. Do not assume that reading the PDF or this file executes the model.

## Model context and units

The technical note describes clear-sky index kappa = measured GHI / clear-sky GHI. Both irradiances must use the same unit, commonly W/m². Night and twilight require an explicit handling rule to avoid division by zero. Estimate correlations and the mean from training data, at a declared regular sampling interval and horizon; do not use future test data.

With S=alpha+K and P=alpha*K, the documented update combines the current and lag-h index and the training mean:

`predicted_kappa_at_t_plus_h = S*kappa(t) - P*kappa(t-h) + (1+P-S)*mean_training_kappa`.

Multiply by clear-sky irradiance at t+h to recover a forecast in irradiance units. This is the mathematical context of the coefficients returned by the lookup. The lookup alone does not compute a clear-sky model, fit correlations, handle nighttime samples or score forecasts.

The generic update relies on wide-sense stationarity; clear-sky normalization aims to approximate it for solar data. The companion solar Algorithm 5 additionally caps predicted kappa by beta before multiplying by the future clear-sky value, and uses kappa=1 below its irradiance threshold epsilon. The unbounded equation above is not the complete solar preprocessing/postprocessing algorithm. The note also discusses removing nighttime samples as an alternative: record which convention and time grid were used. If working with interval irradiation, preserve its energy-per-area unit and interval; do not label it instantaneous irradiance. The note's epsilon is an irradiance/irradiation threshold, not an angle; a solar-elevation filter is a separate option.

## Check against the paper

Extract `K_alpha.zip` into a separate working directory, change MATLAB's current directory there, then run:

```matlab
[K, alpha] = K_alpha(0.4, 0.3, 2);
% Paper (h = 1, rho(1) = 0.4, rho(2) = 0.3, R = 0.05): K = -0.25, alpha = 0.59, so S = 0.34 and P = -0.15
```

The repository `websemantic.json` is a documentation descriptor, not a registered adapter. Do not automatically execute this tool or silently substitute an implementation from another repository.

## Attribution and limits

C. Voyant, G. Notton, J.-L. Duchaud, L. A. Garcia-Gutierrez, J. M. Bright and D. Yang, *Benchmarks for solar radiation time series forecasting*, Renewable Energy 191 (2022), 747–762. https://doi.org/10.1016/j.renene.2022.04.065

Software DOI: https://doi.org/10.5281/zenodo.21755931 . License: MIT. Cite the exact commit or archive used; documentation added later is not automatically included in an earlier DOI archive. No new release is created by these documentation changes. The former execution defect concerned only the distributed function; the article's results are unaffected.
