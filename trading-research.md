# Algorithmic Trading Research

[**← Back to Portfolio Index**](README.md) · [**Security Architecture →**](security-architecture.md) · [**Microsoft Sentinel →**](sentinel.md) · [**AI Agents →**](ai-agents.md) · [**Books & Learning →**](books-and-learning.md)

---

## Scope, Authenticity & Separation from Core Cybersecurity Work

Algorithmic trading is a **separate personal quantitative research area** maintained independently of my core enterprise cybersecurity architecture and Microsoft Sentinel work.

> **Explicit Authenticity & Scope Disclosure:**
> 1. **Historical Exploratory Scripts (`2021–2023`, Private & Archived):** Between 2021 and 2023, I wrote exploratory Python scripts across `Backtest_Codes`, `Functions`, and 13 experimental `archive-*` repositories exploring technical indicators (`RSI + ADX`, `MACD + Renko`, `CPR` compression & value-area relationships, `Heikin-Ashi + Supertrend`, trailing stops) on NSE data. Those historical scripts used simplified bar-close execution assumptions without full slippage/tax modeling, contained legacy broker API configurations, and **are kept permanently Private and Archived**. No historical trading return claims are made.
> 2. **Consolidated Pure-Python Backtesting Prototype (`October 2026`, Published to Private `86sunbot/trading-research` @ `f00ae2a`):** The `trading-research` repository (`src/trading_research/backtest.py`, `indicators.py`, `strategies.py`, and `tests/`, **`9 passed`** regression tests) is an **October 2026 pure-Python prototype** that adapts the core indicator and strategy algorithms from `Backtest_Codes`, `Functions`, `archive-python-utils`, and `archive-trading-strategies` (preserving original third-party credits to `# Author : Mayank Rasu` alongside `@author: sharat.p.surya`) into deterministic, zero-dependency Python functions with next-bar open execution (avoiding look-ahead bias) and explicit transaction-cost deduction (`cost_bps_per_side`). **It is a new software prototype over synthetic bar fixtures and does not represent my historical trading results.**
> 3. **Not Financial Advice:** Nothing in this portfolio constitutes financial, investment, or trading advice.

---

## 1. Consolidated Pure-Python Backtesting & Indicator Prototype (`trading-research` — Refactored October 2026)

- **Visitor Access:** **Public case study with implementation currently private** (`86sunbot/trading-research` on `origin/main` at commit `f00ae2a`, 8 files; held private for Stage D release).

### Problem and Audience
- **Problem:** Ad-hoc technical indicator scripts frequently suffer from look-ahead bias (executing at the same bar close that triggered a signal), rely on unpinned broker SDKs, and omit round-trip transaction costs.
- **Audience:** Python practitioners reviewing clean, self-contained indicator and backtest mechanics.

### Current Capabilities & Source Provenance (`origin/main` @ `f00ae2a`)
- **Core Pivot, RSI & Narrow-CPR Breakout Engine (`src/trading_research/backtest.py`):**
  - `compute_cpr` *(adapted from `Functions:CPR` @ `cb28d48`, `Backtest_Codes:Technical_Indicators.py` @ `b6ea8ce`, & `archive-python-utils:CPR-Width` @ `a42fba0`, `@author: sharat.p.surya`)*: Computes daily Pivot, Bottom Central (`BC`), Top Central (`TC`), and percentage range width (`width_pct`).
  - `compute_rsi` *(adapted from `Backtest_Codes:Technical_Indicators.py` @ `b6ea8ce`, `@author: sharat.p.surya`)*: Standard Wilder-smoothed Relative Strength Index over a configurable lookback period.
  - `run_cpr_breakout_backtest` *(newly implemented in October 2026, inspired by `archive-python-utils:test-open` @ `a70a99c`)*: Evaluates narrow-CPR breakout conditions on bar $t$ close, executes entry on bar $t+1$ open, applies configurable basis-point transaction costs (`cost_bps_per_side`) on both entry and exit, and checks stop-loss thresholds conservatively before bar close.
- **Consolidated Technical Indicators (`src/trading_research/indicators.py`):**
  - `compute_ema` & `compute_macd` *(adapted from `Backtest_Codes:MACD_RENKO_BACKTEST` @ `e275b91` & `renko` @ `558b04c`, preserving original third-party attribution `# Author : Mayank Rasu` & `@author: sharat.p.surya`)*: Exponential Moving Average and MACD (`12/26/9`) line, signal, and histogram.
  - `compute_atr` *(adapted from `Backtest_Codes:MACD_RENKO_BACKTEST` @ `e275b91` & `archive-python-utils:CPR-VR.py` @ `be0560c`, `# Author : Mayank Rasu` & `@author: sharat.p.surya`)*: Wilder's Average True Range.
  - `compute_adx` & `classify_adx_trend_strength` *(adapted from `Functions:Technical_Indicators` @ `701ea64` & `Backtest_Codes:Technical_Indicators.py` @ `b6ea8ce`, `@author: sharat.p.surya`)*: Wilder's `+DI`, `-DI`, Average Directional Index (`ADX`), and 4-bucket trend-strength classifier (`Weak Trend`, `Strong Trend`, `Very Strong Trend`, `Extremely Very Strong Trend`).
  - `compute_heikin_ashi` & `compute_supertrend` *(adapted from `archive-trading-strategies:HA-ST-MFI` @ `4c0d856`, `@author: sharat.p.surya`)*: Recursive Heikin-Ashi smoothed candles and stateful ATR-based Supertrend trailing volatility band (`+1` bullish / `-1` bearish).
  - `build_renko_bricks` *(adapted from `Backtest_Codes:renko` @ `558b04c` & `MACD_RENKO_BACKTEST` @ `e275b91`, `# Author : Mayank Rasu` & `@author: sharat.p.surya`)*: Deterministic fixed-brick price movement construction with $2\times$ reversal threshold.
  - `classify_cpr_relationship` *(adapted from `Functions:CPR` @ `cb28d48` & `archive-python-utils:v2-VR` @ `c985772`, `@author: sharat.p.surya`)*: Seven-state two-session CPR value-area classification (`Higher Value`, `Overlapping Higher Value`, `Lower Value`, `Overlapping Lower Value`, `Inside Value`, `Outside Value`, `Unchanged Value`).
- **Consolidated Rule-Based Strategy Backtesters (`src/trading_research/strategies.py`):**
  - `run_rsi_adx_backtest` *(adapted from `Backtest_Codes:BT-RSI-ADX.py` @ `b6ea8ce` & `archive-trading-strategies:v1-RSI_ADX` @ `a17d143`, `@author: sharat.p.surya`)*: Combines Wilder `RSI` momentum with an `ADX` trend-strength regime filter (`ADX >= threshold` and `+DI > -DI`).
  - `run_resistance_breakout_backtest` *(adapted from `Backtest_Codes:Resistance_Breakout_Backtest` @ `23c85be` & `archive-trading-strategies:v1-Resistance_Breakout` @ `a44f69d`, preserving `# Author : Mayank Rasu` template credit & `@author: sharat.p.surya`)*: Rolling $N$-bar high resistance breakout confirmed by volume expansion (`volume >= multiplier * avg_volume`), executed at bar $t+1$ open with trailing stop-loss and round-trip transaction costs.

### Maturity / Status
- **Consolidated Pure-Python Prototype (`origin/main` @ `f00ae2a` · Root License Selection Pending · Stage D Candidate):** Zero-dependency Python (`dataclasses`) package verified by **9 automated regression and unit tests** run via `pytest` (`9 passed in 0.04s`). Held in **Stage D** until historical OHLCV dataset evaluations and walk-forward notebooks are added.

### Evidence and Limitations
- **Locally Executed Automated Regression Tests:** `PYTHONPATH=src pytest -v tests` -> **`9 passed in 0.04s`** (`4` tests in `test_backtest.py` + `5` regression tests in `test_indicators_and_strategies.py` covering exact hand-calculated reference values, warm-up `None` boundaries, all 7 CPR value-area classifications, next-bar open execution without look-ahead bias, and basis-point transaction cost deduction).
- **Known Limitations:**
  - Currently operates only on synthetic `Bar` sequences in unit tests; does not yet include historical NSE market datasets or live broker connectivity.
  - Bar-level OHLCV simulation cannot resolve intra-bar tick ordering when both a stop-loss and high excursion occur within the same bar.

### Canonical Source & Stage D Plan
- **Access Status:** **Public case study with implementation currently private** (`86sunbot/trading-research` on `origin/main` at commit `f00ae2a`, 8 files — scheduled for Stage D publication once historical dataset evaluations and license selection are finalized).
