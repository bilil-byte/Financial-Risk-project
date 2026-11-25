# Portfolio Risk Engine

A Python notebook that measures and validates the market risk of a multi-asset portfolio: **Value at Risk (VaR)** and **Conditional VaR (CVaR / Expected Shortfall)** with three methods, a **correlation stress test**, and **backtesting** of the model.

The goal is to reproduce, at a small scale, the steps of a risk-management workflow: estimate a potential loss, compare several methods, test how robust the result is when correlations rise, then check afterwards that the model keeps its promises.

> Educational project. Nothing here is investment advice.

## Features

- **Interactive portfolio generator**: the user enters at least 6 tickers, each one is checked against Yahoo Finance (existence, no duplicates), receives a weight between 0 and 1, the weights must sum to 1 and the portfolio must cover at least 3 asset categories.
- **Data**: daily adjusted closing prices from Yahoo Finance (`yfinance`), simple daily returns, average returns, correlation heatmap.
- **VaR and CVaR at 95% and 99%**:
  - *Historical*: empirical quantile of the portfolio's daily returns.
  - *Parametric (variance-covariance)*: normal returns, closed-form VaR and CVaR, with an n-day version (volatility scaled by √n, mean by n).
  - *Monte Carlo*: multivariate geometric Brownian motion, 10,000 correlated paths over 10 days.
- **Correlation stress test**: the correlation matrix is blended with a matrix of ones (coefficient from 0 to 1) while individual volatilities are kept unchanged. VaR/CVaR are recomputed for each coefficient, and the portfolio volatility is decomposed into **risk contributions** per asset (Euler decomposition).
- **Backtesting**:
  - *Kupiec test* (proportion of failures) at 95% and 99%.
  - *Rolling-window backtest* (250 days) of the 95% parametric VaR, with exceptions plotted and tested.

## Example portfolio and results

The notebook was run on a six-asset portfolio over **2017-2022** (1,508 daily returns, including the March 2020 crash and the 2022 sell-off). The assets and weights were chosen for illustration only.

| Asset | Type | Weight |
|---|---|---|
| NVDA | Equity | 30% |
| ^GSPC | Index (S&P 500) | 25% |
| AMZN | Equity | 20% |
| BTC-USD | Cryptocurrency | 10% |
| GLD | ETF (gold) | 10% |
| GC=F | Future (gold) | 5% |

Expected portfolio return: 0.111% per day. Portfolio volatility: 1.71% per day.

**1-day VaR and CVaR**

| Method | VaR 95% | CVaR 95% | VaR 99% | CVaR 99% |
|---|---|---|---|---|
| Historical | 2.66% | 4.07% | 4.86% | 6.56% |
| Parametric | 2.71% | 3.42% | 3.88% | 4.46% |

**10-day VaR and CVaR**

| Method | VaR 95% | CVaR 95% | VaR 99% | CVaR 99% |
|---|---|---|---|---|
| Parametric (σ√10, mean ×10) | 7.80% | 10.07% | 11.50% | 13.33% |
| Monte Carlo (10,000 paths) | 8.09% | 10.06% | 11.35% | 12.93% |

**Backtesting**

| Test | Result |
|---|---|
| Kupiec, 95% (in-sample) | 4.71% exceptions (5% expected), LR = 0.28: not rejected |
| Kupiec, 99% (in-sample) | 1.72% exceptions (1% expected), LR = 6.57: rejected |
| Rolling 250-day window, 95% | 89 exceptions out of 1,258 days (7.07% vs 5% expected), LR = 10.16, p-value = 0.0014: rejected |

### Key findings

- **The normal-based VaR is reliable at 95% but not at 99%.** The historical 99% VaR (4.86%) is 25% above the parametric one (3.88%), and the Kupiec test rejects the model at 99%. The worst day (-11.9% on 12 March 2020) lies about 7 standard deviations from the mean.
- **CVaR reveals the fat tails that VaR hides.** The historical CVaR/VaR ratio (1.53 at 95%) is well above the normal-distribution ratio (about 1.25). Monte Carlo and parametric methods share the same normality assumption, so Monte Carlo mainly adds a flexible horizon.
- **Diversification is worth about a quarter of the risk.** Moving from the observed correlations to perfect correlation raises the 1-day 95% VaR from 2.71% to 3.68% (+36%). Bitcoin accounts for about 40% of that increase while weighing 10% of the portfolio: its low correlations with the other assets hide a very high volatility.
- **A 250-day rolling window reacts slowly to regime changes.** Exceptions cluster in 2018, March 2020 and 2022, and the VaR stays too wide for about a year after the 2020 crash.

## Repository content

```
.
├── Risk_Engine_nb.ipynb   # the full project (generator, VaR/CVaR, stress test, backtests)
└── README.md
```

## Getting started

Requirements: Python 3.10+ and the libraries below.

```bash
pip install yfinance pandas numpy matplotlib seaborn scipy jupyter
```

Open the notebook and run all cells:

```bash
jupyter notebook Risk_Engine_nb.ipynb
```

The first cells ask for the number of tickers, then each ticker (Yahoo Finance symbol, e.g. `AMZN`, `BTC-USD`, `GC=F`, `^GSPC`) and its weight. To reproduce the example above, enter the six assets and weights from the table.

Notes on reproducibility:
- The Monte Carlo simulation uses a fixed seed (42).
- Yahoo Finance data can be revised, so figures may differ slightly from those above.

## Assumptions and limitations

The notebook ends with a detailed "Assumptions & limitations" section. The main points:

- Returns are assumed i.i.d. and normal for the parametric and Monte Carlo methods (no fat tails, skewness or volatility clustering), with a constant covariance matrix and constant weights.
- The Kupiec test is in-sample, which favours the model. It checks the frequency of exceptions, not their independence.
- The correlation stress test only changes correlations: a coefficient of 1 is a theoretical bound, and volatilities are not stressed.
- The "asset categories" required by the generator are Yahoo Finance instrument types, not risk factors (GLD and GC=F both track gold).

## Possible extensions

- EWMA volatility and GARCH models, which react faster to volatility changes
- Fat-tailed distributions (Student-t) or filtered historical simulation
- Christoffersen independence test and backtesting of the CVaR

## Author

- Lilian NKWEMFO
- Cédric Victor BILONG