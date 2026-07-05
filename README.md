[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1153wBB8uYrviFaOUHtB5r4UdTr-VcFpb#scrollTo=ZsY5O-qnOXRJ)

# var-es-engine
Portfolio Value-at-Risk and Expected Shortfall Measurement System implementing Historical, Parametric, and Monte Carlo methods with Kupiec backtesting
[VaR_ES_Engine_Project_Blueprint.md](https://github.com/user-attachments/files/29677503/VaR_ES_Engine_Project_Blueprint.md)

# Value-at-Risk & Expected Shortfall Engine — Full Project Blueprint

## 1. Project Overview
A production-style market risk engine that measures potential portfolio losses using three
industry-standard methodologies — **Historical Simulation**, **Parametric (Variance-Covariance)**,
and **Monte Carlo Simulation** — for both **Value-at-Risk (VaR)** and **Expected Shortfall (ES / CVaR)**.
The engine ingests real market data, cleans and transforms it into return series, computes risk
metrics at multiple confidence levels, and statistically validates model accuracy through a
**Kupiec Proportion-of-Failures backtest** — the same test regulators and bank model-validation
teams use to certify VaR models.

## 2. Real-World Finance Use Case
- **Bank Market Risk Departments:** Compute daily VaR/ES for regulatory capital under Basel III / FRTB (Fundamental Review of the Trading Book).
- **Hedge Funds & Prop Desks:** Set daily risk limits per trading book and firm-wide.
- **Asset Managers:** Report portfolio tail-risk exposure to clients and investment committees.
- **Risk Committees:** Use backtesting results to certify (or flag for review) internal risk models, exactly as required by regulators.

## 3. System Architecture
```
                ┌──────────────────────┐
                │   Yahoo Finance API  │
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │    MarketDataLoader  │   (fetch + validate raw prices)
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │      DataCleaner     │   (NaN handling, log returns)
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │       VaREngine      │  (Historical / Parametric / Monte Carlo)
                └──────────┬───────────┘
                           │
         ┌─────────────────┼─────────────────────┐
         │                                       │
┌────────▼───────────┐                ┌──────────▼─────────────┐
│  VaRBacktester     │                │   Visualization Laye   │
│  (Kupiec POF test) │                │   (matplotlib charts)  │
└────────┬───────────┘                └────────────┬───────────┘
         │                                         │
         └────────────────┬────────────────────────┘
                          │
                ┌─────────▼───────────┐
                │  Consolidated Report│  (CSV + console dashboard)
                └─────────────────────┘
```

## 4. Required APIs and Data Sources
| Source | Purpose | Notes |
|---|---|---|
| **Yahoo Finance** (`yfinance`) | Daily adjusted close prices for portfolio constituents | Free, no API key, primary source used in the notebook |
| **FRED** (optional upgrade) | Risk-free rate for Sharpe-adjusted risk reporting | Free with API key |
| **Alpha Vantage** (optional) | Intraday data for shorter VaR horizons | Free tier is rate-limited |
| **Polygon** (optional, institutional) | Tick-level data for intraday VaR | Paid for full history |

## 5. Required Python Libraries
```
numpy, pandas, yfinance, matplotlib, scipy, dataclasses (stdlib), typing (stdlib)
```
Optional upgrades would add: `arch` (GARCH modeling), `copulas` (tail dependence), `ipywidgets` (interactive dashboard).

## 6. Folder/File Structure (Conceptual — even though built in Colab)
```
var-es-engine/
│
├── VaR_ES_Engine.ipynb          # Main Colab notebook (all logic, cell-by-cell)
├── var_es_summary.csv           # Auto-generated output: risk metrics table
├── README.md                    # Project overview, setup, and usage instructions
├── requirements.txt             # pip-installable dependency list
└── /assets                      # (optional) exported chart images for README/GitHub
```

## 7. Step-by-Step Build Guide
1. Install/import dependencies and set global plotting style.
2. Define portfolio configuration (tickers, weights, notional value, confidence levels).
3. Validate configuration (weights sum to 1, no negative weights, positive notional).
4. Fetch historical adjusted close prices via `yfinance`.
5. Clean data: forward-fill small gaps, drop remaining NaNs, filter invalid prices.
6. Compute log returns per asset, then aggregate into a weighted portfolio return series.
7. Build the `VaREngine` class implementing Historical, Parametric, and Monte Carlo VaR/ES.
8. Run all three methods across all configured confidence levels into a summary table.
9. Build the `VaRBacktester` class: rolling VaR forecasts + Kupiec POF statistical test.
10. Visualize: return distribution with VaR/ES overlays, rolling backtest with breach markers, cross-method comparison bars, cumulative return/drawdown chart.
11. Generate a consolidated console risk report and export results to CSV.
12. Document limitations and upgrade paths for future iterations.

## 8. Data Collection Pipeline
- `MarketDataLoader.fetch()` pulls adjusted close prices for all configured tickers in a single batched `yf.download()` call (efficient, avoids N separate API calls).
- Handles both single-ticker and multi-ticker MultiIndex column structures returned by `yfinance`.
- Raises descriptive errors on empty responses and warns on partially missing tickers rather than failing silently.

## 9. Data Cleaning & Feature Engineering
- Forward-fills isolated missing values (e.g., holiday mismatches across exchanges), then drops any remaining incomplete rows.
- Filters out any non-positive prices that would corrupt log-return calculations.
- Computes **log returns** (`ln(P_t / P_t-1)`) rather than simple returns — the standard convention in risk modeling because log returns are time-additive across horizons.
- Aggregates individual asset returns into a single weighted portfolio return series using the configured static weights.

## 10. Core Models/Algorithms
- **Historical Simulation VaR/ES:** Empirical quantile of realized returns — model-free, captures actual historical fat tails.
- **Parametric VaR/ES:** Closed-form Gaussian formulas, including the analytical Expected Shortfall formula for a normal distribution (`ES = μ - σ·φ(z)/(1-c)`).
- **Monte Carlo VaR/ES:** Simulates 100,000+ correlated return scenarios by drawing from a multivariate normal distribution fitted to the historical covariance matrix, capturing diversification effects across assets.
- **Kupiec Proportion-of-Failures Test:** A likelihood-ratio test comparing observed VaR breach frequency against the theoretically expected frequency, producing a chi-squared p-value used to accept/reject model calibration.

## 11. Visualizations & Dashboard Components
- Return distribution histogram with fitted normal overlay and VaR/ES threshold lines.
- Rolling VaR forecast vs. actual returns time series with breach points highlighted.
- Side-by-side bar chart comparing VaR and ES magnitudes across all three methodologies and confidence levels.
- Cumulative portfolio growth chart paired with a drawdown chart for risk context.

## 12. Performance Metrics
- VaR and ES in both percentage and dollar terms at 95% and 99% confidence.
- Kupiec LR statistic, p-value, and pass/fail calibration flag per confidence level.
- Observed vs. expected breach counts and breach rate.
- Maximum historical drawdown as a supplementary risk context metric.

## 13. Final Deliverables
- A fully executable Colab notebook (`VaR_ES_Engine.ipynb`) implementing the full pipeline end-to-end.
- A CSV export of the consolidated risk summary table.
- Four professional-grade chart sets suitable for a risk report or GitHub README.
- A console-printed, formatted Portfolio Value-at-Risk and Expected Shortfall Measurement System risk report.

## 14. Resume Description
> *"Built a Portfolio Value-at-Risk and Expected Shortfall Measurement System in Python, implementing Historical Simulation, Parametric, and Monte Carlo methodologies across a multi-asset portfolio; validated model accuracy using a Kupiec Proportion-of-Failures backtest and produced automated risk reporting and visualization consistent with Basel/FRTB market risk practices."*

## 15. Potential Upgrades
- EWMA (RiskMetrics) or GARCH(1,1) conditional volatility for time-varying VaR.
- Student-t copula Monte Carlo to capture fat tails and joint tail dependence.
- Cornish-Fisher VaR adjustment for skewness/kurtosis.
- Component/Incremental VaR for per-position risk attribution.
- Christoffersen independence test to detect breach clustering (not just frequency).
- Interactive `ipywidgets` dashboard for live parameter tuning inside Colab.
