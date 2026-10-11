# Algorithmic Trading Research

[**← Back to Portfolio Index**](README.md) · [**Security Architecture →**](security-architecture.md) · [**Microsoft Sentinel →**](sentinel.md) · [**AI Agents →**](ai-agents.md) · [**Books & Learning →**](books-and-learning.md)

---

## Scope, Authenticity & Separation from Core Cybersecurity Work

Algorithmic trading is a **separate personal quantitative research area** maintained independently of my core enterprise cybersecurity architecture and Microsoft Sentinel work.

> **Explicit Authenticity & Scope Disclosure:**
> 1. **Historical Exploratory Scripts (`2021–2023`, Private & Archived):** Between 2021 and 2023, I wrote exploratory Python scripts across `Backtest_Codes` and 13 experimental repositories exploring technical indicators (`RSI + ADX`, `MACD + Renko`, `CPR` compression, trailing stops) on NSE data. Those historical scripts used simplified bar-close execution assumptions without full slippage/tax modeling, contained legacy broker API configurations, and **are kept permanently Private and Archived**. No historical trading return claims are made.
> 2. **New Minimal Backtest Prototype (`October 2026`, Stage D Candidate):** The local `trading-research` repository (`/Users/suryap/.gemini/antigravity/scratch/trading-research`) contains a **newly written October 2026 pure-Python prototype** (`src/trading_research/backtest.py`, `139 lines`, `4 passed` unit tests) created to demonstrate clean indicator calculation (`compute_cpr`, `compute_rsi`), next-bar open execution (avoiding look-ahead bias), and explicit transaction-cost deduction (`cost_bps_per_side`). **It is a new software prototype over synthetic bar fixtures and does not represent my historical trading results.**
> 3. **Not Financial Advice:** Nothing in this portfolio constitutes financial, investment, or trading advice.

---

## 1. New Minimal Backtesting Prototype (`trading-research` — Created October 2026)

- **Visitor Access:** **Public case study with implementation currently private** (`86sunbot/trading-research` is currently private and held for Stage D release).

### Problem and Audience
- **Problem:** Ad-hoc technical indicator scripts frequently suffer from look-ahead bias (executing at the same bar close that triggered a signal) and omit round-trip transaction costs.
- **Audience:** Python practitioners reviewing clean, self-contained backtest mechanics.

### Current Capabilities (Documented in Case Study)
- **Central Pivot Range (`CPR`) Calculation (`compute_cpr`):** Computes daily Pivot, Bottom Central (`BC`), Top Central (`TC`), and percentage range width (`width_pct`).
- **Wilder's Relative Strength Index (`compute_rsi`):** Standard smoothed RSI calculation over a configurable lookback period.
- **Next-Bar Open Breakout Backtester (`run_cpr_breakout_backtest`):** Evaluates narrow-CPR breakout conditions on bar $t$ close, executes entry on bar $t+1$ open, applies configurable basis-point transaction costs (`cost_bps_per_side`) on both entry and exit, and checks stop-loss thresholds conservatively before bar close.

### Maturity / Status
- **New Minimal Prototype (Created October 2026 · Existing Root `MIT` License Preserved · Stage D Candidate):** Pure-Python (`dataclasses`) module verified by **4 automated unit tests** run via `pytest` (`4 passed in 0.01s`). Held in **Stage D** until historical OHLCV dataset evaluations and walk-forward notebooks are added.

### Evidence and Limitations
- **Locally Executed Automated Tests:** `PYTHONPATH=src pytest -v tests` -> `4 passed in 0.01s` (`test_compute_cpr_levels_and_width`, `test_compute_rsi_wilder_smoothing`, `test_cpr_breakout_backtest_executes_next_bar_and_deducts_costs`, `test_cpr_breakout_stop_loss_triggers`).
- **Known Limitations:**
  - Currently operates only on synthetic `Bar` sequences in unit tests; does not yet include historical NSE market datasets or multi-strategy comparisons.
  - Bar-level OHLCV simulation cannot resolve intra-bar tick ordering when both a stop-loss and high excursion occur within the same bar.

### Canonical Source & Stage D Plan
- **Access Status:** **Public case study with implementation currently private** (`86sunbot/trading-research`, committed locally at `f1e50ed` — scheduled for Stage D publication once historical dataset evaluations are added).
