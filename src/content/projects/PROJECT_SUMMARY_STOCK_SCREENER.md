---
title: "Stock Screener"
description: "A simple interactive stock screener built with yfinance and matplotlib. Shows price action with moving averages and RSI for any ticker, with a live text box to switch symbols."
tech: ["Python", "Matplotlib", "Pandas", "yfinance"]
github_link: "https://github.com/articwnd/Stock-Screener"
year: 2026
featured: true
---
# Stock Screener

A simple interactive stock screener built with `yfinance` and `matplotlib`. Shows price action with moving averages and RSI for any ticker, with a live text box to switch symbols.

## Features

- Pulls 1 year of daily historical data via Yahoo Finance
- Plots closing price with 20-day and 50-day SMAs
- Plots 14-period RSI with overbought (70) / oversold (30) zones shaded
- Text box to enter a new ticker, charts update on submit
- Basic error handling for invalid tickers or empty data

## Requirements

- Python 3.9+
- `yfinance`
- `pandas`
- `matplotlib`

```bash
pip install yfinance pandas matplotlib
```

## Usage

```bash
python main.py
```

A window opens showing MSFT by default (v0.1.1). Type a new ticker in the "Ticker:" box and press Enter to reload both charts.

## Project Structure

- `main.py` - main app (`StockScreener` class)
- `search.py` - prototype ticker search/autocomplete bar (work in progress, not yet wired into `main.py`)

## Known Issues / TODO

- RSI formula's no-loss edge case (`loss == 0`) needs review, currently returns 0 instead of 100
- `search.py` suggestion-filtering logic is commented out and incomplete
- Every ticker change triggers a fresh 1y download, no caching
