# FX-trading-bot-EUR-USD
A EUR/USD trading bot using various quant and technical strategies layered together for the Asia/LDN/NY sessions
## Notebook overview (CRISP-DM framing)
The accompanying `notebook.ipynb` follows the CRISP-DM lifecycle:

- **Business Understanding:** Defines the goal of building a systematic, ML-assisted EUR/USD strategy for a small account, prioritising low drawdown, risk control, and scalability across Asia/London/NY sessions.
- **Data Understanding:** Explores intraday EUR/USD price action by hour and session, visualising volatility patterns, range distributions, and economic-event overlays to uncover session-dependent behaviour.
- **Data Preparation:** Drops low-signal columns, recomputes technical features, engineers a noise-banded classification target, and sets up preprocessing plus time-aware train/test splits.
- **Modelling:** Trains XGBoost classifiers (with Optuna tuning) and regime-aware variants (KMeans/HDBSCAN) and wires runs into MLflow for comparison.
- **Evaluation:** Assesses classification performance, per-session/regime lift, and strategy-level PnL from simple trading rules to ensure the modelling edge is tradable.
- **Deployment:** Outlines how the modelling pipeline can feed an automated trading bot once robustness criteria are met.

## Repository structure
- `notebook.ipynb` – end-to-end CRISP-DM workflow for the EUR/USD strategy.
- `data/`
  - `usd-eur.xml` – historical EUR/USD price data used for EDA, feature engineering, and backtesting.
  - `economic calendar dataset/` – economic-event data for session/news overlays.
- `images/` – supporting visuals (e.g., candlestick chart in the notebook).
- `mlruns/` – MLflow tracking artifacts for model comparisons.
- `regime_xgb_hdb_equity_curve.png` – equity curve for the regime-aware XGB strategy.
- `TECHNICAL_FEATURES.md` – documentation of engineered indicators/feature set.

## Evaluation metrics (from the notebook)
- **Classification metrics:** Test accuracy around 51–53% and ROC AUC ≈ 0.55–0.58 for baseline and regime-aware XGB models, reflecting a modest but tradable edge in noisy intraday FX data.
- **Risk/return metrics:** Profit factor targets above 1.2–1.5, risk-adjusted ratios (Sharpe/Sortino) above 1.0–1.5, and maximum drawdown kept below ~15–25% to prioritise capital protection.
- **Session/regime diagnostics:** AUC and confusion-matrix breakdowns by session/regime to verify London/NY edge versus lower-volatility Asia periods, plus simple strategy-level PnL to validate that modelling lift translates to trading performance.
