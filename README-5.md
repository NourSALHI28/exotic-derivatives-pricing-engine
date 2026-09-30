# Exotic Derivatives Pricing & Heston Calibration Engine

A Python project for **volatility surface analysis, Heston stochastic volatility calibration, Monte Carlo simulation and exotic derivatives pricing**.

It implements an end-to-end quantitative workflow, from option market inputs to the valuation and risk analysis of path-dependent structured products. The main case study is a **1-year equity Autocall**, valued under both **Geometric Brownian Motion (GBM)** and a **calibrated Heston model**.

---

## Key results

| Result | Value |
|---|---:|
| Heston calibration RMSE (option-price units) | **≈ €0.27** |
| Autocall value under GBM | €1,004.34 |
| Autocall value under Heston | €991.40 |
| Difference (Heston − GBM) | **−€12.93 (−1.29% of notional)** |

**Takeaway:** moving from constant volatility to stochastic volatility materially changes the value of a path-dependent product, because it changes the distribution of paths, the probability of early redemption and the probability of downside scenarios.

> Results are based on a **synthetic** implied-volatility surface (see [Limitations](#limitations)).

---

## Workflow

```text
Option / volatility data
        ↓
Data preparation
        ↓
Implied-volatility surface
        ↓
Heston calibration
        ↓
Monte Carlo simulation (GBM & Heston)
        ↓
Exotic derivatives pricing (Asian, Barrier, Autocall)
        ↓
GBM vs Heston comparison
        ↓
Risk & sensitivity analysis
```

---

## Quick start

```bash
git clone https://github.com/NourSALHI28/exotic-derivatives-pricing-engine.git
cd exotic-derivatives-pricing-engine
pip install -r requirements.txt
jupyter notebook notebooks/
```

Run the tests:

```bash
python -m pytest tests/
```

---

## Models

### Black-Scholes

European call price:

$$
C = S_0 N(d_1) - K e^{-rT} N(d_2)
$$

$$
d_1 = \frac{\ln(S_0/K) + \left(r + \tfrac{1}{2}\sigma^2\right)T}{\sigma\sqrt{T}}, \qquad d_2 = d_1 - \sigma\sqrt{T}
$$

The Black-Scholes engine is also used to convert implied-volatility observations into option prices for calibration.

### Implied-volatility surface

The demonstration surface covers:

- **Strikes:** 80, 90, 100, 110, 120
- **Maturities:** 1M, 3M, 6M, 1Y, 2Y

It captures the fact that implied volatility is not constant across strike and maturity (smile and term structure).

### Heston stochastic volatility

$$
dS_t = r S_t\,dt + \sqrt{v_t}\,S_t\,dW_t^S
$$

$$
dv_t = \kappa(\theta - v_t)\,dt + \xi\sqrt{v_t}\,dW_t^v, \qquad dW_t^S\,dW_t^v = \rho\,dt
$$

| Parameter | Interpretation |
|---|---|
| $v_0$ | Initial variance |
| $\kappa$ | Mean-reversion speed |
| $\theta$ | Long-run variance |
| $\xi$ | Volatility of variance |
| $\rho$ | Spot / variance correlation |

### Calibration

European prices under Heston are computed with the **characteristic function and numerical integration**. Parameters are estimated by minimising the mean squared pricing error:

$$
\min_{\Theta} \; \frac{1}{N}\sum_{i=1}^{N}\left(P_i^{\text{Heston}} - P_i^{\text{Market}}\right)^2, \qquad \Theta = (\kappa, \theta, \xi, \rho, v_0)
$$

### Monte Carlo engine

**GBM:**

$$
S_{t+\Delta t} = S_t \exp\!\left[\left(r - \tfrac{1}{2}\sigma^2\right)\Delta t + \sigma\sqrt{\Delta t}\,Z\right]
$$

**Heston:** price and variance are simulated jointly with a **full-truncation Euler** scheme, which prevents negative variance from entering the diffusion term.

---

## Exotic products

### Arithmetic Asian call

$$
\text{Payoff} = \max\!\left(\frac{1}{N}\sum_{t=1}^{N} S_t - K,\; 0\right)
$$

Outputs: Monte Carlo price, standard error, 95% confidence interval, payoff distribution, volatility sensitivity.

### Up-and-out barrier call (discretely monitored)

$$
\text{Payoff} =
\begin{cases}
\max(S_T - K, 0) & \text{if } \max_t S_t < H \\
0 & \text{otherwise}
\end{cases}
$$

Outputs: price, knockout and survival probabilities, barrier sensitivity, confidence intervals, visualisation of surviving vs knocked-out paths.

### Equity Autocall

| Parameter | Value |
|---|---:|
| Initial underlying | 100 |
| Notional | €1,000 |
| Maturity | 1 year |
| Observations | Quarterly |
| Autocall barrier | 100% |
| Annual coupon | 8% |
| Capital protection barrier | 60% |
| Risk-free rate | 3% |

- At each quarterly date, if $S_t \geq 100\% \, S_0$, the note is redeemed at notional plus accrued coupon.
- At maturity, if $S_T \geq 60\% \, S_0$, the notional is repaid.
- Otherwise the investor bears the loss: $\text{Redemption} = N \, S_T / S_0$.

**Risk analytics:** present value, Monte Carlo standard error, 95% confidence interval, early-redemption probability (total and by observation date), expected product life, probability of terminal capital loss, distribution of cash flows.

> Probabilities are **risk-neutral** and should not be read as real-world forecasts.

---

## Visualisations

3D implied-volatility surface · market vs Heston prices · calibration errors · GBM and Heston paths · Autocall GBM vs Heston · early-redemption and capital-loss probabilities · expected product life · Asian payoff distribution · barrier path analysis.

![Volatility surface](results/volatility_surface.png)
![GBM vs Heston](results/gbm_vs_heston.png)

---

## Repository structure

```text
exotic-derivatives-pricing-engine/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
│   ├── 01_volatility_surface.ipynb
│   ├── 02_heston_calibration.ipynb
│   ├── 03_monte_carlo.ipynb
│   ├── 04_asian_option.ipynb
│   ├── 05_barrier_option.ipynb
│   └── 06_autocall_gbm_vs_heston.ipynb
├── src/
│   ├── black_scholes.py
│   ├── implied_volatility.py
│   ├── heston.py
│   ├── calibration.py
│   ├── monte_carlo.py
│   ├── exotic_options.py
│   └── autocall.py
├── tests/
│   ├── test_black_scholes.py
│   ├── test_heston.py
│   └── test_autocall.py
└── results/
    ├── volatility_surface.png
    ├── heston_calibration.png
    ├── gbm_vs_heston.png
    └── redemption_probabilities.png
```

**Stack:** Python · NumPy · pandas · SciPy · Matplotlib

---

## Limitations

- The implied-volatility surface is **synthetic**; it demonstrates the calibration and pricing architecture, not market-consistent prices.
- Barrier monitoring is **discrete**; no Brownian Bridge correction yet.
- No variance-reduction techniques yet, so Monte Carlo errors are larger than necessary.

## Roadmap

1. **Real option-chain data:** bid/ask sourcing, mid prices, liquidity filters, arbitrage and data-quality checks.
2. **Market implied-volatility surface:** IV extraction, moneyness, interpolation, smile/skew analysis.
3. **Better calibration:** liquidity-weighted and bid/ask-aware objectives, calibration in IV space, parameter stability.
4. **Advanced Monte Carlo:** antithetic variates, Brownian Bridge, convergence analysis.
5. **Structured-product risk:** Greeks, spot/vol/coupon/barrier sensitivities, scenario analysis.
6. **Scale:** larger option datasets, multiple valuation dates, historical calibration.

---

## Disclaimer

Educational and research project. Models are simplified and results are not investment advice, executable prices or production-grade valuations.

## Author

**Nour Salhi** · Quantitative finance, derivatives & risk
[LinkedIn](https://www.linkedin.com/in/nour-salhi-finance) · [GitHub](https://github.com/NourSALHI28)
