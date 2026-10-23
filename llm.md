# ARTU: documentation and execution status

## Authority and scope

This describes the repository at source commit `11aefa42049c737c1e6d845a5428d29aa48dd506`. The repository provides a technical note and `K_alpha.zip`, containing a MATLAB coefficient lookup function and four tables. It is not a complete executable forecasting pipeline. This documentation does not alter the archived scientific source.

## Known execution blocker

**The archived `K_alpha.m` declares input `Q` but tests `R`, which is not defined in its function scope.** Executing `[K,alpha]=K_alpha(0.6,0.4,2)` from the extracted archive with MATLAB R2025b produces `MATLAB:UndefinedFunction`, "Unrecognized function or variable 'R'." The extracted function was compared byte-for-byte with the ZIP member. No coefficients from this call are valid results, and no numerical output should be invented. A code correction and separate scientific validation are required before advertising this archive as directly executable.

## Intended inputs and outputs

- `corrh`, `corr2h`: real, finite, dimensionless autocorrelations at forecast lags h and 2h of the same training clear-sky-index series. The source comment states -1 < correlation < 1; the stored interpolation grid spans -1:0.05:1. Grid coverage alone does not certify feasibility of an arbitrary pair of autocorrelations.
- `Q`: intended discrete selector 0, 1, 2, 3 for tables R0, R0.01, R0.05, R0.1. It is not a continuous percentage or an irradiance value. Do not silently choose a reliability level for the user. Its mapping is stated in the source comments but is blocked by the undefined-R defect.
- Intended outputs `K`, `alpha`: dimensionless interpolated coefficients. Once execution is repaired, table validity, finite values and edge behavior still need checking. The current function implements no explicit validation of input bounds or missing data.

The four `.mat` files must be resolvable by MATLAB when the lookup is called. Do not assume that reading the PDF or this file executes the model.

## Model context and units

The technical note describes clear-sky index kappa = measured GHI / clear-sky GHI. Both irradiances must use the same unit, commonly W/m². Night and twilight require an explicit handling rule to avoid division by zero. Estimate correlations and the mean from training data, at a declared regular sampling interval and horizon; do not use future test data.

With S=alpha+K and P=alpha*K, the documented update combines the current and lag-h index and the training mean:

`predicted_kappa_at_t_plus_h = S*kappa(t) - P*kappa(t-h) + (1+P-S)*mean_training_kappa`.

Multiply by clear-sky irradiance at t+h to recover a forecast in irradiance units. This is the intended mathematical context, not an executable result from the blocked archive. The lookup alone does not compute a clear-sky model, fit correlations, handle nighttime samples or score forecasts.

## Reproduce the blocker

Extract `K_alpha.zip` into a separate working directory, change MATLAB's current directory there, then run:

```matlab
[K, alpha] = K_alpha(0.6, 0.4, 2);
% Current archive: undefined variable R; no valid coefficients returned.
```

The repository `websemantic.json` is a documentation descriptor with execution blocked, not a registered adapter. Do not automatically execute this tool or silently substitute an implementation from another repository.

## Attribution and limits

C. Voyant, G. Notton, J.-L. Duchaud, L. A. Garcia-Gutierrez, J. M. Bright and D. Yang, *Benchmarks for solar radiation time series forecasting*, Renewable Energy 191 (2022), 747–762. https://doi.org/10.1016/j.renene.2022.04.065

Software DOI: https://doi.org/10.5281/zenodo.21755931 . License: MIT. Cite the exact commit or archive used; documentation added later is not automatically included in an earlier DOI archive. No new release is created by these documentation changes. The execution defect is evidence about the distributed function, not evidence that the article's scientific conclusions are false.
