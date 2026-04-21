# Quant A – Public API Reference

This document describes all public functions exposed by the `app/quant_a` module.
It is intended for **Quant B** (portfolio module) to reuse strategies, data, and metrics
without depending on the Streamlit UI layer.

> **Rule:** Quant B must never import from `ui_quant_a.py`.

---

## Data Formats

### Price series

All strategy functions accept a `prices` argument of type:

```python
prices: pandas.Series
  index : pandas.DatetimeIndex  # trading dates
  values: float                 # closing prices
```

### Strategy DataFrame (`strat_df`)

All strategy functions return a `pandas.DataFrame` with at least:

| Column | Type | Description |
|---|---|---|
| `price` | float | Closing price |
| `position` | float | Position in [-1, +1] |
| `strategy_returns` | float | Period return of the strategy (decimal) |
| `equity_curve` | float | Cumulative value, starting at 1.0 |

Additional columns vary by strategy (see below).

### Trade DataFrame (`trades_df`)

Returned by `extract_trades_from_position`:

| Column | Type | Description |
|---|---|---|
| `entry_date` | datetime | Trade entry date |
| `exit_date` | datetime | Trade exit date |
| `direction` | str | `"LONG"` or `"SHORT"` |
| `entry_price` | float | Entry price |
| `exit_price` | float | Exit price |
| `holding_period_bars` | int | Duration in number of bars |
| `trade_return` | float | Decimal return (e.g. 0.05 = +5 %) |
| `trade_return_pct` | float | Return in percent (e.g. 5.0 = +5 %) |

---

## Module: `data_loader.py`

### `load_history`

```python
load_history(
    ticker: str,
    start: str | None = None,
    end: str | None = None,
    interval: str = "1d",
    intraday_days: int = 5,
) -> pd.DataFrame
```

Generic Yahoo Finance data loader. Returns a DataFrame with OHLCV columns indexed by date.

- `ticker`: Yahoo Finance symbol (e.g. `"^FCHI"`, `"AAPL"`, `"EURUSD=X"`)
- `start` / `end`: date strings `"YYYY-MM-DD"` (ignored for intraday)
- `interval`: `"1d"`, `"1wk"`, `"1mo"`, `"5m"`, `"15m"`, `"60m"`
- `intraday_days`: history depth for intraday intervals

### `load_cac40_history`

```python
load_cac40_history(
    start: str | None = None,
    end: str | None = None,
    interval: str = "1d",
    intraday_days: int = 5,
) -> pd.DataFrame
```

Convenience wrapper for the CAC 40 (`^FCHI`). Same return format as `load_history`.

### `get_last_cac40_close`

```python
get_last_cac40_close() -> float
```

Returns the latest available closing price of the CAC 40.

---

## Module: `strategies.py`

### `buy_and_hold`

```python
buy_and_hold(prices: pd.Series) -> pd.DataFrame
```

Passive strategy: fully invested at all times (`position = 1.0`).
Returns the standard `strat_df`.

### `moving_average_crossover`

```python
moving_average_crossover(
    prices: pd.Series,
    short_window: int = 50,
    long_window: int = 200,
) -> pd.DataFrame
```

Trend-following strategy:
- `position = 1.0` when short MA > long MA
- `position = 0.0` otherwise

Additional columns: `ma_short`, `ma_long`.

### `regime_switch_trend_meanrev`

```python
regime_switch_trend_meanrev(
    prices: pd.Series,
    vol_short_window: int = 20,
    vol_long_window: int = 100,
    alpha: float = 1.0,
    trend_ma_window: int = 50,
    mr_window: int = 20,
    z_threshold: float = 1.0,
) -> pd.DataFrame
```

Two-regime strategy:
- **TREND** regime (short vol > α × long vol): follows the trend via moving average signal
- **MR** regime: contrarian mean-reversion via z-score signal

Additional columns: `regime`, `vol_short`, `vol_long`, `ma_trend`, `mr_mu`, `mr_sigma`, `zscore`, `trend_signal`, `mr_signal`.

### `extract_trades_from_position`

```python
extract_trades_from_position(
    prices: pd.Series,
    position: pd.Series,
) -> pd.DataFrame
```

Reconstructs the full trade history from a price series and a position series (`-1`, `0`, `+1`).
Returns a `trades_df` (see format above).

---

## Module: `metrics.py`

### `compute_all_metrics`

```python
compute_all_metrics(
    equity_curve: pd.Series,
    returns: pd.Series,
    risk_free_rate: float = 0.0,
    periods_per_year: int = 252,
) -> dict
```

Returns a dict with:

```python
{
    "total_return": float,
    "annualized_return": float,
    "annualized_volatility": float,
    "sharpe_ratio": float,
    "max_drawdown": float,   # negative value, e.g. -0.15 = -15%
}
```

### `compute_trade_metrics`

```python
compute_trade_metrics(trades_df: pd.DataFrame) -> dict
```

Returns a dict with:

```python
{
    "n_trades": int,
    "win_rate": float,
    "pct_longs": float,
    "pct_shorts": float,
    "avg_trade_return": float,
    "avg_win_return": float,
    "avg_loss_return": float,
    "avg_holding_period": float,
}
```

---

## Module: `universe.py`

### `get_asset_classes`

```python
get_asset_classes() -> list[str]
```

Returns the list of available asset classes: `["Indices", "Forex", "Equities", "Commodities"]`.

### `get_assets_by_class`

```python
get_assets_by_class(asset_class: str) -> list[Asset]
```

Returns the list of `Asset` objects for a given class.
Each `Asset` has: `name` (str), `ticker` (str), `asset_class` (str).

### `get_default_asset`

```python
get_default_asset() -> Asset
```

Returns the default asset (CAC 40).

---

## Quick integration example (Quant B)

```python
from app.quant_a.data_loader import load_history
from app.quant_a.strategies import moving_average_crossover, extract_trades_from_position
from app.quant_a.metrics import compute_all_metrics, compute_trade_metrics

# Load data
df = load_history(ticker="^FCHI", start="2020-01-01", interval="1d")
prices = df["Close"]

# Run strategy
strat_df = moving_average_crossover(prices, short_window=20, long_window=100)

# Compute metrics
metrics = compute_all_metrics(
    equity_curve=strat_df["equity_curve"],
    returns=strat_df["strategy_returns"],
    periods_per_year=252,
)

# Extract trades
trades_df = extract_trades_from_position(strat_df["price"], strat_df["position"])
trade_metrics = compute_trade_metrics(trades_df)
```
