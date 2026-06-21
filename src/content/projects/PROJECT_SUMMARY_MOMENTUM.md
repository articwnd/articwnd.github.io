---
title: "Cross-Sectional Momentum Strategy"
description: "Backtested 12-1 month cross-sectional momentum strategy on the top 50 S&P 500 constituents by index weight, with monthly rebalancing and equal-weighted decile selection."
tech: ["Python", "Pandas", "NumPy", "yfinance", "Matplotlib"]
github_link: "https://github.com/articwnd/Momentum-Trading"
year: 2026
featured: true
---

## Problem

Momentum, the tendency for assets with strong recent performance to continue
outperforming over intermediate horizons, is one of the most persistent and
heavily documented anomalies in empirical finance. But the term gets used
loosely. A moving-average crossover on a single stock is trend-following, not
momentum, and conflating the two leads to a strategy that is neither
rigorously specified nor comparable to the academic literature it claims to
implement.

A correct momentum implementation requires cross-sectional ranking across a
universe of assets, a specific formation period, and a mechanism to avoid
well-documented short-term reversal effects, none of which a single-asset
trend signal provides.

## Approach

Built a Python backtesting pipeline (`momentum.py`) implementing the
Jegadeesh & Titman (1993) 12-1 month formation framework:

- **Universe:** top 50 S&P 500 constituents by index weight, sourced from
  SlickCharts.
- **Data layer:** `yfinance` daily adjusted close, resampled to month-end for
  monthly rebalancing.
- **Signal construction:** for each stock at each rebalance date, the trailing
  12-month return is calculated with the most recent month skipped (the "12-1"
  convention), since returns in the final month before formation are
  empirically prone to short-term reversal rather than continuation.
- **Portfolio construction:** stocks ranked by signal each month; the top 10
  (top quintile of the 50-stock universe) are selected and equally weighted.
  No short leg in the current implementation.
- **Lookahead control:** portfolio weights computed from the prior month's
  signal are shifted forward one period before being applied to that month's
  realized returns, so the backtest never trades on information not yet
  available at the rebalance date.
- **Benchmark:** equal-weight buy-and-hold across all 50 tickers, computed
  alongside the strategy for direct comparison.
- **Performance metrics:** annualized Sharpe ratio, total return, and maximum
  drawdown, computed for both strategy and benchmark.

## Tradeoffs

**Long-only, no short leg.** The academic factor is long-short (top decile
minus bottom decile), which isolates the momentum premium from broad market
beta. The current long-only version is simpler to reason about and avoids
short-borrow assumptions, but its returns are conflated with general market
exposure. Adding the short leg is the natural next step and is a small code
change given the ranking logic already exists, the open question is whether
to model it as a funded short (subtract borrow cost) or a notional overlay.

**Equal weighting over signal-proportional weighting.** Equal weighting
across the top N is simple and avoids letting one extreme momentum reading
dominate the portfolio. The tradeoff is that it discards information, two
stocks with very different momentum strength get the same weight if both
clear the top-10 cutoff. Signal-weighted (or rank-weighted) allocation is a
documented alternative not yet implemented.

**Current top-50 universe over point-in-time constituents.** Sourcing tickers
from today's S&P 500 weights is simple and matches the heatmap project's
universe for consistency across the portfolio. The cost is survivorship bias:
the backtest implicitly assumes foreknowledge of which 50 companies would be
index leaders in 2026, when run over a 2015-2026 window. This is a known,
documented limitation rather than an oversight, and a point-in-time
constituent list (e.g. from a paid vendor) would be required to remove it.

**No transaction costs or slippage.** Monthly rebalancing with a 10-stock
portfolio implies real turnover, and ignoring costs overstates the strategy's
realistic return. Not yet modeled; flagged as a known gap rather than
addressed.

**Monthly rebalancing over weekly or daily.** Matches the academic convention
and limits turnover, but is also the convention every introductory momentum
implementation defaults to, so it has not yet been tested against alternative
holding periods to see whether monthly is actually optimal for this specific
universe, as opposed to merely standard.

## Outcome

A working cross-sectional backtest that replaces the earlier single-stock
trend-following script with a correctly specified, ranked, multi-asset
momentum implementation, including lookahead-safe weight shifting and
benchmark comparison.

This project is less mature than the correlation heatmap: the rebalancing
mechanics have not yet been stress-tested against edge cases (a sparse-data
month producing fewer than `TOP_N` valid signals is handled, but not yet
verified against a real occurrence in the data), and several of the tradeoffs
above are open design questions rather than finalized decisions. The natural
next steps, in rough priority order, are: adding the short leg, modeling
transaction costs, and parameter sensitivity testing across formation period,
skip period, and portfolio size, several of which are referenced in posts on
aligrithm.com on cost-aware ranking and signal calibration.
