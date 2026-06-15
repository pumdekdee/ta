# Claude Project Context: ta

This file provides context for the **ta** repository, used as part of an AI trading-knowledge project.

## Project Overview
* **Name**: ta (Technical Analysis Library in Python)
* **Language**: Python (pandas + NumPy only — pure Python, no C dependencies)
* **Type**: Technical analysis indicator library
* **Purpose**: Compute technical analysis indicators and features directly on pandas DataFrames/Series — a pure-Python alternative to `TA-Lib` that avoids the C-library installation requirement.
* **License**: MIT

## Core System Boundaries
* **Pandas-native**: every indicator function takes pandas `Series` (e.g. `close`, `high`, `low`, `volume`) and returns a pandas `Series` — output aligns by index with the input DataFrame, so results can be assigned directly as new columns.
* **No compiled dependencies**: unlike `TA-Lib`, installation is a plain `pip install` with no system-level C library required — useful when `TA-Lib`'s build step is a blocker.
* **Two usage styles**:
  * **Per-indicator classes/functions** (`ta.trend.SMAIndicator`, `ta.momentum.RSIIndicator`, etc.) — instantiate then call `.sma_indicator()` / `.rsi()` etc.
  * **Bulk helper** (`ta.add_all_ta_features`) — adds a large set of standard indicators to a DataFrame in one call, commonly used for feature engineering before feeding ML models.

## Repository Structure & Key Directories
* `/ta/` - Core package, organized by indicator category:
  * `/ta/trend.py` - Trend indicators (SMA, EMA, MACD, ADX, Ichimoku, etc.)
  * `/ta/momentum.py` - Momentum indicators (RSI, Stochastic, Williams %R, ROC, etc.)
  * `/ta/volatility.py` - Volatility indicators (Bollinger Bands, ATR, Keltner Channel, etc.)
  * `/ta/volume.py` - Volume-based indicators (OBV, MFI, VWAP, Accumulation/Distribution, etc.)
  * `/ta/others.py` - Misc return/cumulative-return helpers
  * `/ta/utils.py` - Internal helper functions
  * `/ta/wrapper.py` - `add_all_ta_features` bulk-feature function
* `/examples/` - Example scripts showing DataFrame-based usage
* `/docs/` - Indicator reference documentation

## Standard Operational Commands
* **Install**: `pip install ta`

## Standard Python API Usage
```python
import pandas as pd
from ta.momentum import RSIIndicator
from ta.trend import MACD
from ta.volatility import BollingerBands
from ta import add_all_ta_features

df = pd.DataFrame({"open": ..., "high": ..., "low": ..., "close": ..., "volume": ...})

# Per-indicator usage
df["rsi"] = RSIIndicator(close=df["close"], window=14).rsi()
macd = MACD(close=df["close"])
df["macd"] = macd.macd()
df["macd_signal"] = macd.macd_signal()

bb = BollingerBands(close=df["close"], window=20, window_dev=2)
df["bb_high"] = bb.bollinger_hband()
df["bb_low"] = bb.bollinger_lband()

# Bulk feature engineering (adds ~80+ indicator columns)
df = add_all_ta_features(
    df, open="open", high="high", low="low", close="close", volume="volume",
    fillna=True,
)
```

## AI Assistant Guidelines for this Project
1. **Pandas-first**: always work with DataFrame columns directly; output Series share the input's index, so assigning `df["indicator_name"] = ...` is the standard pattern.
2. **`TA-Lib` alternative**: when a user hits installation issues with `TA-Lib` (missing C library), suggest `ta` as a drop-in-concept (though not identical API) alternative since it's pure Python.
3. **`add_all_ta_features` for ML pipelines**: when the user is doing feature engineering for ML models (e.g. for `FinRL` or `machine-learning-for-trading` workflows), `add_all_ta_features` is a fast way to generate a broad feature set — but warn about multicollinearity among the resulting indicators and the need for feature selection.
4. **`fillna` parameter**: indicators near the start of a series produce `NaN` until enough data points exist (e.g. a 20-period SMA needs 20 rows) — most functions accept `fillna=True/False`; explain the tradeoff (filled values vs. dropping early rows) when relevant.
5. **Column naming**: when adding multiple indicators, recommend consistent, descriptive column names (e.g. `rsi_14`, `macd_12_26_9`) to avoid collisions, especially when combining with other libraries like `ta-lib-python`.
