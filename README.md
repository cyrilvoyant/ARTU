# ARTU — Statistical Reference Forecasting Methods for Solar Radiation

Reference implementation and technical documentation for **ARTU**, a second-order autoregressive naive benchmark introduced for solar irradiance forecasting, together with four classical reference methods it is built alongside: Persistence, Climatology, CLIPER, and Exponential Smoothing. Companion resource for:

> C. Voyant, G. Notton, J.-L. Duchaud, L. A. Garcia-Gutierrez, J. M. Bright, D. Yang, "Benchmarks for solar radiation time series forecasting," *Renewable Energy*, vol. 191, pp. 747-762, 2022. https://doi.org/10.1016/j.renene.2022.04.065 ([arXiv:2203.14959](https://arxiv.org/abs/2203.14959))

## Contents

- **`ARTU.pdf`** — "Statistical Reference Method" technical note (algorithmic pseudo-code for all five reference methods below).
- **`K_alpha.zip`** — MATLAB implementation of the ARTU-specific interpolation step: `K_alpha.m` plus four precomputed lookup tables `R0.mat`, `R0.01.mat`, `R0.05.mat`, `R0.1.mat` (one per measurement-reliability level $R$).
- **`LICENSE`** — MIT.

## The methods (from `ARTU.pdf`)

All five operate on the clear-sky index $\kappa(t)=I_{GH}(t)/I_{CS}(t)$ (global horizontal irradiance normalized by a clear-sky model), with $\kappa=1$ substituted whenever $I_{CS}(t)$ falls below a solar-elevation threshold $\varepsilon$ (night/twilight):

- **PER (Persistence)**: propagates the last valid clear-sky index forward, $\widehat{I}_{GH}(t+h)=\min\big(I_{GH}(t-n)\cdot I_{CS}(t+h)/I_{CS}(t-n),\ \gamma\cdot I_{CS}(t+h)\big)$, clipped by an over-irradiance factor $\gamma\in[1,2]$.
- **CLIM (Climatology)**: forecasts using the mean clear-sky index over the whole training sample, $\widehat\kappa=\overline\kappa$ — a naive floor that any useful model should beat.
- **CLIPER (Climatology-Persistence)**: blends the two above via $\rho=\mathrm{ACF}(\kappa(n),\kappa(n-h))$, $\widehat\kappa=\rho\,\kappa(t-h)+(1-\rho)\,\overline\kappa$.
- **ES (Exponential Smoothing)**: like CLIPER, but averages the last $h_{\max}\in[10,48]$ clear-sky index values with exponentially decaying weights $(1-\rho)^i$ instead of just the lag-$h$ value.
- **ARTU**: a second-order extension of CLIPER/ES. It uses both the lag-$h$ and lag-$2h$ autocorrelations, $\rho_1=\mathrm{ACF}(\kappa(n),\kappa(n-h))$ and $\rho_2=\mathrm{ACF}(\kappa(n),\kappa(n-2h))$, and looks up the coefficients $(\alpha,K)$ solving the corresponding second-order AR system by **bilinear interpolation** over a precomputed grid $M(R)$ indexed by $(\rho_1,\rho_2)$, at a measurement-reliability level $R\in\{0,0.01,0.05,0.1\}$ (`K_alpha.m` + the `R0*.mat` tables in `K_alpha.zip`). With $S=\alpha+K$ and $P=\alpha K$:

$$\widehat\kappa(t)=S\cdot\kappa(t)-P\cdot\kappa(t-h)+(1+P-S)\cdot\overline\kappa,\qquad \widehat I_{GH}(t+h)=\widehat\kappa(t)\cdot I_{CS}(t+h)$$

This ARTU update rule (and the `K_alpha` lookup table interpolation it depends on) is also used as-is by the GHI reference forecasters in [Make_Stationary](https://github.com/cyrilvoyant/Make_Stationary/blob/main/Forecasting/K_alpha.m).

## Usage

```matlab
% Unzip K_alpha.zip into your MATLAB path, then given the lag-h and lag-2h
% clear-sky-index autocorrelations (corrh, corr2h) and a reliability level Q:
[K, alpha] = K_alpha(corrh, corr2h, Q);   % Q in {0,1,2,3} -> R in {0, 0.01, 0.05, 0.1}
```

See `ARTU.pdf`, Algorithm 5, for the full ARTU update using `K`/`alpha`.

## Citation

If you use this code or method, please cite the paper above (see [`citation.cff`](citation.cff)).

## License

MIT. See [`LICENSE`](LICENSE).
