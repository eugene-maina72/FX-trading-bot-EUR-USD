# Technical Feature Engineering Overview

This note captures the exact operations used to build the technical analysis feature set before tackling the market behaviour questions listed in the **Analytical Questions** section of the notebook.

## Data Inputs
- **5-minute EUR/USD bars (60 days)** via `yfinance` → core intraday set for feature building and answering questions 1–3.
- **1-minute EUR/USD bars (7 days)** → microstructure checks and sanity validation of indicators on higher-resolution data.
- **Daily EUR/USD bars (2 years)** → long-horizon context and volatility baselines.
- **Economic calendar** filtered to EUR/USD only (`data/economic calendar dataset/relevant_events.csv`) → used for event-based filters.

## Pre-processing Steps
1. Load each dataset and **force `Datetime` index** (`pd.to_datetime`, sort index).
2. **Drop duplicates** and remove any rows with missing OHLC values.
3. **Forward-fill minor gaps** inside a session; skip creating bars that do not exist in the source feed.
4. Normalize timestamps to **UTC** and add local hour for session tagging.
5. Keep only price columns (`Open`, `High`, `Low`, `Close`); ignore Yahoo volume (mostly zero for FX).

## Core Feature Construction (5-minute set)
- **Price geometry**
  - `mid_price = (High + Low) / 2`
  - `hl_range = High - Low`, `body = Close - Open`, upper/lower wick sizes
  - `return_1 = Close.pct_change()`, `log_ret_1 = ln(Close/Close.shift(1))`
- **Volatility**
  - `true_range`, `ATR_14` / `ATR_21` (TA-Lib) for absolute and normalized range
  - Rolling return std (`vol_20`, `vol_50`) to proxy realized volatility
  - `range_pct = hl_range / Close` for scale-free intraday range sizing
- **Trend**
  - EMAs (`ema_9`, `ema_21`, `ema_55`) and slopes; `sma_20`, `sma_50`
  - `MACD`, `signal`, `hist` plus histogram slope for momentum/trend persistence
  - `ADX_14` with `+DI` / `-DI` for trend strength vs. chop
- **Mean-reversion / breakout tension**
  - `rsi_14` and `rsi_2` (short-term stretch)
  - Bollinger z-score (`(Close - bb_mid_20) / bb_std_20`), band width, and %B
  - Stochastic %K/%D (14,3) and `CCI_20` for overbought/oversold context
  - Donchian channel distances (`donchian_20_high_dist`, `donchian_20_low_dist`) to flag breakout potential
  - Keltner/Bollinger **bandwidth vs. ATR** to quantify volatility compression
- **Time/session context**
  - `hour`, `day_of_week`, and **session label** (Asia/London/NY) based on hour boundaries
  - `is_session_opening` flag for the first 2–3 bars of each session
- **Event risk filters**
  - For each bar, flag `is_high_impact_now` and `is_high_impact_next_30m` using `relevant_events.csv`
  - These flags can be used to exclude or down-weight bars around news releases

## How These Features Answer the Market Behaviour Questions
- **Q1 (Volatility by hour/session):** use `ATR_14`, `true_range`, `range_pct`, and rolling std; aggregate by `session` and `hour`.
- **Q2 (Mean reversion vs. breakouts/trends):** compare RSI/Bollinger/Stochastic stretches to MACD/EMA/ADX trend strength; Donchian distances and band compression help isolate breakout setups.
- **Q3 (Typical ranges/holding periods):** use `hl_range`, `range_pct`, and ATR-normalized moves to size realistic targets/stops for small accounts; session/time features contextualize which windows support wider vs. tighter ranges.

## Notes for Reuse
- Primary feature table is built on the **5-minute dataset**; replicate on 1-minute data only for validation or microstructure-specific experiments.
- TA-Lib is used wherever possible to avoid hand-rolled indicator drift; all lookbacks above assume standard defaults but can be parameterized for Optuna sweeps.
- Keep the feature frame aligned (no future leakage): indicators use only current/past bars, and any forward return/label should be created after the feature columns are locked.
