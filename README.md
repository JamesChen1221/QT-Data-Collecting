# US Stock Day-Trading Model — Training Data Collector

This is a small side project focused on collecting data to train a quantitative day trading model for the US stock market.

The pipeline automatically fetches historical price data from Yahoo Finance, computes technical indicators, and writes results back to a structured Excel database. Each row represents one trade entry, identified by a ticker and an opening date.

---

## 1. Project Purpose

To systematically collect pre-trade features for each trade entry and use them as training data for a machine learning classifier focused on US equity intraday trading.

---

## 2. Development Approach

### Data Collection
All market data is sourced from **Yahoo Finance** via the `yfinance` library (free tier), using daily OHLCV data for historical indicators and 1-minute intraday data for same-day price behavior.

### Features Computed

| Feature | Description |
|---|---|
| 6-Month RSI / ADX Sequence | 120-day RSI(14) and ADX(14) series before the trade |
| Price Distance (5d / 1m / 6m) | % distance of prior close from the N-day high and low |
| Prior Close | Closing price of the day before the opening date |
| 20-Day Avg Trading Value | Average of (Close × Volume) over the prior 20 trading days |
| 120-Day Close Sequence | Raw closing price series for the prior 120 trading days |
| Opening Price | First-minute open of the regular trading session |
| 10-Min Low / 30-Min High / 90-Min High | Intraday price behavior after open |
| Low Before High | Lowest price between minute 10 and the 90-min high point |

All features are computed using only data available **before** the opening date to prevent data leakage. The pipeline is incremental — existing values are never overwritten, so the script can be re-run safely at any time.

### AI-Assisted Development
This project was developed with the assistance of **Kiro**, an AI-powered development environment, used throughout for feature logic design, debugging, and refactoring. All generated code was reviewed and validated against real market data.

---

## 3. Usage

### Prerequisites

```bash
pip install -r requirements.txt
```

### Excel Setup

The pipeline reads from and writes to `量化交易.xlsx`, which must contain a sheet named `資料庫` with at least two columns: `開盤日期` (opening date) and `公司代碼` (ticker symbol). All other feature columns are optional — the script detects which ones exist and fills in only empty cells.

### Running

**Double-click** `執行指標計算.bat`, or run from the command line:

```bash
python calculate_indicators.py
```

---

## 4. Additional Information

### Project Structure

```
├── calculate_indicators.py   # Main pipeline script
├── 執行指標計算.bat            # Windows one-click runner
├── 量化交易.xlsx               # Trade database (gitignored)
├── 新產業分類建議.xlsx          # Proposed industry classification reference
├── requirements.txt
└── README.md
```

### Notes

- **Intraday data** is only available for the **last 7 days** due to yfinance free-tier limits. Older rows are skipped for those fields.
- **New stocks** with limited history will have price distance computed using all available data rather than the full requested window.
- All sequences are ordered **oldest to most recent**: `[day_N, ..., day_1]`.
- RSI > 70: overbought / RSI < 30: oversold — ADX > 25: strong trend / ADX < 20: ranging.

### Data Source

[Yahoo Finance](https://finance.yahoo.com) via [`yfinance`](https://github.com/ranaroussi/yfinance) (free tier).
