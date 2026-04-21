# Quant Dashboard – Systematic Strategy Backtester

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-1.x-FF4B4B?logo=streamlit&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Branch](https://img.shields.io/badge/branch-quant__a-blueviolet)
![Data](https://img.shields.io/badge/Data-Yahoo%20Finance-6001D2?logo=yahoo&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

A **Streamlit-based quantitative finance dashboard** for backtesting systematic trading strategies on financial markets. Developed as part of a 4th-year engineering project at ESILV.

---

## Project Context

This project was built for the **Python, Git & Linux** course at **ESILV (École Supérieure d'Ingénieurs Léonard-de-Vinci)**, 4th year (A4).

The repository is organized into two independent modules, developed on separate branches:

| Branch | Module | Description |
|---|---|---|
| `quant_a` | **Quant A** | Univariate backtesting engine (this branch) |
| `quant_b` | **Quant B** | Multi-asset portfolio management |

---

## Main Features

### Quant A – Univariate Backtesting Engine

- **Multi-asset support** across four classes:
  - *Indices*: CAC 40, DAX 40, S&P 500
  - *Forex*: EUR/USD, GBP/USD, USD/JPY
  - *Equities*: TotalEnergies, LVMH, Apple, Microsoft
  - *Commodities*: Gold (futures), WTI Crude Oil
- **Three trading strategies**:
  - **Buy & Hold** — fully-invested passive benchmark
  - **Moving Average Crossover** — signal-based trend following (configurable short/long windows)
  - **Regime Switching (Trend + Mean-Reversion)** — volatility-based regime detection combining a trend-following and a mean-reversion signal
- **Automatic parameter optimization** via grid search (MA windows, regime thresholds)
- **Performance metrics**: total return, annualized return, annualized volatility, Sharpe ratio, max drawdown
- **Trading metrics**: win rate, long/short split, average trade return, average holding period
- **Trade history table** with CSV export
- **ML module** (optional): next-step return prediction and log-linear price trend projection
- **Daily automated reports** — cron-compatible CAC 40 report generator
- **Candlestick price explorer** with adjustable display horizons (1M → MAX) at daily, weekly, monthly, and intraday resolution

### Quant B – Multi-Asset Portfolio *(in progress)*

- Equal-weight and custom-weight portfolio construction
- Portfolio value simulation with configurable rebalancing (daily / weekly / monthly)
- Correlation matrix heatmap and diversification ratio
- Asset-level and portfolio-level annualized volatility

---

## Tech Stack

| Layer | Technology |
|---|---|
| Dashboard | [Streamlit](https://streamlit.io/) |
| Market data | [yfinance](https://github.com/ranaroussi/yfinance) (Yahoo Finance) |
| Data manipulation | [pandas](https://pandas.pydata.org/), [NumPy](https://numpy.org/) |
| Visualization | [Plotly](https://plotly.com/python/) |
| Language | Python 3.10+ |

---

## Installation

```bash
# 1. Clone the repository on the Quant A branch
git clone -b quant_a https://github.com/MaximeFARRE/Projet-Python-Git-A4.git
cd Projet-Python-Git-A4

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt
```

---

## Usage

### Launch the dashboard

```bash
streamlit run main.py
```

Open your browser at `http://localhost:8501`.

**Sidebar controls:**
- Select an asset class and a specific asset
- Choose a data frequency (daily, weekly, monthly, intraday)
- Set the backtest date range
- Pick a strategy and adjust its parameters
- Click **Optimize** to run automatic grid-search optimization

### Automate daily reports with cron (Linux / macOS)

The `daily_report.py` module generates a structured text report for the CAC 40. Add the following line to your crontab (`crontab -e`) to run it every weekday at 18:00:

```
0 18 * * 1-5 cd /path/to/project && /path/to/.venv/bin/python -m app.quant_a.daily_report >> logs/cron.log 2>&1
```

Reports are saved to `reports/quant_a/daily_report_YYYY-MM-DD.txt` and can also be viewed directly in the dashboard under the **Daily reports** section.

---

## Repository Structure

```
.
├── main.py                        # Streamlit entry point
├── requirements.txt               # Python dependencies
├── .streamlit/
│   └── config.toml                # Dark theme configuration
├── app/
│   ├── quant_a/                   # Quant A – Univariate backtesting engine
│   │   ├── data_loader.py         # Yahoo Finance data loading (generic + CAC 40)
│   │   ├── universe.py            # Asset universe definition (indices, forex, equities…)
│   │   ├── strategies.py          # Buy & Hold, MA Crossover, Regime Switching
│   │   ├── metrics.py             # Performance and trade-level metrics
│   │   ├── optimizers.py          # Grid-search parameter optimization
│   │   ├── ml.py                  # ML price forecasting (linear regression)
│   │   ├── daily_report.py        # Automated daily report generator
│   │   └── ui_quant_a.py          # Streamlit UI for Quant A
│   └── quant_b/                   # Quant B – Multi-asset portfolio
│       ├── portfolio.py           # Portfolio value computation and rebalancing
│       ├── metrics.py             # Portfolio and per-asset metrics
│       └── page_quant_b.py        # Streamlit UI for Quant B
├── reports/                       # Generated daily reports (gitignored)
└── docs/
    └── api_quant_a.md             # Public API reference for Quant A
```

---

## Screenshots

*Screenshots will be added here once the application is running.*

<!-- To add a screenshot:
1. Run `streamlit run main.py`
2. Take a screenshot and save it to screenshots/
3. Replace this comment with: ![Dashboard](screenshots/dashboard.png)
-->

---

## Contributors

| Name | Module | GitHub |
|---|---|---|
| Maxime Farré | Quant A | [@MaximeFARRE](https://github.com/MaximeFARRE) |
| Émilien Combaret | Quant B | [@EmilienCombaret](https://github.com/EmilienCombaret) |

---

## Limitations

- **Quant B integration**: the `quant_b` module is present but not yet fully integrated with `main.py` on this branch. It is developed on the `quant_b` branch and requires a merge to run end-to-end.
- **No transaction costs**: the backtester does not model bid-ask spreads or commissions. All results are gross of fees.
- **Intraday data depth**: Yahoo Finance limits intraday data to the last 5–7 calendar days depending on the interval.
- **In-sample optimization**: the parameter grid search runs on the full selected period. No walk-forward or out-of-sample validation is performed.
- **ML forecasting**: the linear regression model is illustrative only and is not intended for production trading signals.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
