# Exotic Derivatives Pricing & Heston Calibration Engine

A Python-based quantitative finance project for **volatility surface analysis, Heston model calibration, Monte Carlo simulation, and exotic derivatives pricing**.

The project implements an end-to-end quantitative workflow from option market inputs to the valuation and risk analysis of path-dependent structured products.

The main case study is a **1-year equity Autocall**, valued under both **Geometric Brownian Motion (GBM)** and a **calibrated Heston stochastic volatility model**.

---

## Project Overview

The objective of this project is to reproduce a simplified version of a quantitative derivatives workflow:

```text
Option / Volatility Data
        ↓
Data Preparation
        ↓
Volatility Surface
        ↓
Heston Calibration
        ↓
Monte Carlo Simulation
        ↓
Exotic Derivatives Pricing
        ↓
GBM vs Heston Comparison
        ↓
Risk & Sensitivity Analysis
```

The project focuses particularly on the relationship between **volatility modelling and the valuation of path-dependent exotic derivatives**.

---

## Key Features

### Black-Scholes Pricing

Implementation of the Black-Scholes model for European call and put options:

\[
C = S_0N(d_1)-Ke^{-rT}N(d_2)
\]

with:

\[
d_1 =
\frac{\ln(S_0/K)+(r+\frac{1}{2}\sigma^2)T}
{\sigma\sqrt{T}}
\]

and:

\[
d_2=d_1-\sigma\sqrt{T}
\]

The Black-Scholes engine is also used to transform implied-volatility observations into option prices for calibration.

---

## Volatility Surface

The framework supports volatility observations across multiple:

- strikes;
- maturities;
- moneyness levels.

The current demonstration uses a synthetic volatility surface covering:

```text
Strikes:
80, 90, 100, 110, 120

Maturities:
1M, 3M, 6M, 1Y, 2Y
```

The volatility surface allows the model to capture the fact that market implied volatility is not constant across strike and maturity.

> **Current limitation:** the demonstration surface is synthetic. A planned extension of the project is to source and clean real option-chain bid/ask data and construct the implied-volatility surface directly from market observations.

---

## Heston Stochastic Volatility Model

Unlike Black-Scholes/GBM, which assumes constant volatility, the Heston model assumes that variance follows its own stochastic process.

The underlying follows:

\[
dS_t = rS_tdt+\sqrt{v_t}S_tdW_t^S
\]

while variance evolves according to:

\[
dv_t =
\kappa(\theta-v_t)dt
+
\xi\sqrt{v_t}dW_t^v
\]

with:

\[
dW_t^S dW_t^v=\rho dt
\]

The calibrated parameters are:

| Parameter | Interpretation |
|---|---|
| \(v_0\) | Initial variance |
| \(\kappa\) | Mean-reversion speed |
| \(\theta\) | Long-run variance |
| \(\xi\) | Volatility of variance |
| \(\rho\) | Spot/variance correlation |

---

## Heston Calibration

European option prices are calculated under Heston using the model's characteristic function and numerical integration.

The model parameters are estimated by minimizing the discrepancy between market and model option prices:

\[
\min_{\Theta}
\frac{1}{N}
\sum_{i=1}^{N}
\left(
P_i^{Heston}-P_i^{Market}
\right)^2
\]

where:

\[
\Theta =
(\kappa,\theta,\xi,\rho,v_0)
\]

The current calibration produced:

```text
Heston Calibration RMSE: 0.272898
```

This corresponds to an RMSE of approximately **€0.27 in option-price units** on the synthetic calibration dataset.

---

## Monte Carlo Engine

The project contains Monte Carlo simulation engines for both:

### Geometric Brownian Motion

\[
S_{t+\Delta t}
=
S_t
\exp
\left[
\left(r-\frac{1}{2}\sigma^2\right)\Delta t
+
\sigma\sqrt{\Delta t}Z
\right]
\]

### Heston Stochastic Volatility

Both the underlying price and stochastic variance processes are simulated.

The implementation uses a **full-truncation Euler approach** to handle the variance process and prevent negative simulated variance from being used in the diffusion term.

---

# Exotic Derivatives

The framework includes Monte Carlo pricing functionality for path-dependent derivatives.

## Asian Option

The arithmetic Asian call payoff is:

\[
\max
\left(
\frac{1}{N}
\sum_{t=1}^{N}S_t-K,
0
\right)
\]

The project calculates:

- Monte Carlo price;
- standard error;
- 95% confidence interval;
- payoff distribution;
- volatility sensitivity.

---

## Barrier Option

The project implements a discretely monitored **Up-and-Out Call**.

Its payoff is:

\[
\text{Payoff}=
\begin{cases}
\max(S_T-K,0), & \max_t S_t < H \\
0, & \max_t S_t \ge H
\end{cases}
\]

The analysis includes:

- option valuation;
- knockout probability;
- survival probability;
- barrier sensitivity;
- Monte Carlo confidence intervals;
- visualization of surviving and knocked-out paths.

A future extension is to implement **Brownian Bridge corrections** for continuous-barrier monitoring.

---

# Autocall Structured Product

The main exotic product studied in this project is a simplified **single-underlying equity Autocall**.

### Product Characteristics

| Parameter | Value |
|---|---:|
| Initial underlying | 100 |
| Notional | €1,000 |
| Maturity | 1 year |
| Observation frequency | Quarterly |
| Autocall barrier | 100% |
| Annual coupon | 8% |
| Capital protection barrier | 60% |
| Risk-free rate | 3% |

At each quarterly observation date, the product checks whether:

\[
S_t \geq 100\%S_0
\]

If the condition is satisfied, the note is redeemed and pays the notional plus the accrued coupon.

If the product survives until maturity and:

\[
S_T \geq 60\%S_0
\]

the principal is repaid.

If:

\[
S_T < 60\%S_0
\]

the investor participates in the decline of the underlying:

\[
\text{Redemption}
=
N\frac{S_T}{S_0}
\]

---

# GBM vs Heston Autocall Valuation

The same Autocall payoff is valued using both market models.

The latest simulation produced:

| Model | Autocall Value |
|---|---:|
| GBM | €1,004.34 |
| Heston | €991.40 |
| Difference | **−€12.93** |

The Heston valuation is approximately **1.29% of notional lower** than the GBM valuation in this experiment.

This demonstrates that the choice of volatility model can materially affect the valuation of path-dependent derivatives.

GBM assumes constant volatility, whereas Heston incorporates:

- stochastic variance;
- volatility mean reversion;
- spot-volatility correlation;
- volatility skew dynamics;
- non-constant future volatility.

These features modify the distribution of future underlying paths and therefore the probability of early redemption and downside scenarios.

---

# Risk Analytics

The Autocall engine calculates:

- theoretical present value;
- Monte Carlo standard error;
- 95% confidence interval;
- probability of early redemption;
- redemption probability by observation date;
- expected product life;
- probability of terminal capital loss under the risk-neutral simulation;
- distribution of product cash flows.

> Risk-neutral probabilities generated by the pricing model should not be interpreted as real-world forecasts of investor outcomes.

---

# Visualizations

The project generates several quantitative visualizations:

- 3D implied-volatility surface;
- market vs Heston calibrated option prices;
- Heston calibration errors;
- GBM simulated paths;
- Heston simulated paths;
- Autocall valuation comparison;
- early-redemption probabilities;
- capital-loss probabilities;
- expected product life;
- Asian option payoff distribution;
- barrier-option path analysis.

---

# Technology Stack

The project is implemented entirely in Python.

Main libraries:

```text
NumPy
Pandas
SciPy
Matplotlib
```

Main quantitative techniques:

```text
Black-Scholes
Implied Volatility
Volatility Surface Analysis
Numerical Root Finding
Numerical Integration
Optimization
Heston Calibration
Monte Carlo Simulation
Stochastic Volatility
Path-Dependent Option Pricing
Structured Product Valuation
```

---

# Repository Structure

```text
exotic-derivatives-engine/
│
├── README.md
├── requirements.txt
│
├── data/
│   ├── raw/
│   └── processed/
│
├── notebooks/
│   ├── 01_volatility_surface.ipynb
│   ├── 02_heston_calibration.ipynb
│   ├── 03_monte_carlo.ipynb
│   ├── 04_asian_option.ipynb
│   ├── 05_barrier_option.ipynb
│   └── 06_autocall_gbm_vs_heston.ipynb
│
├── src/
│   ├── black_scholes.py
│   ├── implied_volatility.py
│   ├── heston.py
│   ├── calibration.py
│   ├── monte_carlo.py
│   ├── exotic_options.py
│   └── autocall.py
│
├── tests/
│   ├── test_black_scholes.py
│   ├── test_heston.py
│   └── test_autocall.py
│
└── results/
    ├── volatility_surface.png
    ├── heston_calibration.png
    ├── gbm_vs_heston.png
    └── redemption_probabilities.png
```

---

# Installation

Clone the repository:

```bash
git clone <repository-url>
cd exotic-derivatives-engine
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

Example `requirements.txt`:

```text
numpy
pandas
scipy
matplotlib
jupyter
```

---

# Roadmap

The current version establishes the quantitative pricing and calibration architecture.

Planned extensions include:

1. **Real option-chain market data**
   - bid/ask sourcing;
   - mid-price construction;
   - volume and open-interest filters;
   - spread analysis;
   - arbitrage and data-quality checks.

2. **Real implied-volatility surface**
   - implied-volatility extraction;
   - moneyness transformation;
   - interpolation;
   - smile/skew analysis.

3. **Improved Heston calibration**
   - liquidity-weighted calibration;
   - bid/ask-aware objective functions;
   - calibration by implied volatility;
   - parameter stability analysis.

4. **Advanced Monte Carlo**
   - variance reduction;
   - antithetic variables;
   - Brownian Bridge;
   - convergence analysis.

5. **Structured-product analytics**
   - Greeks;
   - spot sensitivity;
   - volatility sensitivity;
   - coupon sensitivity;
   - barrier sensitivity;
   - scenario analysis.

6. **Large option datasets**
   - scalable data-cleaning pipeline;
   - thousands of option observations;
   - multiple valuation dates;
   - historical calibration analysis.

---

# Disclaimer

This project is intended for **educational, research and portfolio purposes**.

The models and results are simplified implementations and should not be interpreted as investment advice, executable market prices, or production-grade valuation outputs.

The current volatility dataset is synthetic and is used to demonstrate the calibration and pricing architecture.

---

## Author

**Nour Salhi**

Finance & Quantitative Finance  
Python | Derivatives | Risk | Asset Management
