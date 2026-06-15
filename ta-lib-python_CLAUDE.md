# Claude Project Context: ta-lib-python

This file provides codebase context for the **ta-lib-python** repository, used as part of an AI trading-knowledge project.

## Project Overview
* **Name**: ta-lib-python
* **Language**: Python (Cython wrapper around the TA-Lib C library)
* **Type**: Technical analysis indicator library
* **Purpose**: Provide fast, vectorized implementations of 150+ technical indicators (moving averages, oscillators, pattern recognition, volatility/volume indicators) for use with NumPy/pandas data.
* **License**: BSD-style (permissive)

## Core System Boundaries
* **Hard dependency on TA-Lib C library**: this Python package is a wrapper — the underlying `ta-lib` C library (or `ta-lib-c` via pip on newer setups) must be installed/available before `pip install TA-Lib` will build successfully.
* **Two API styles**:
  * **Function API** (`talib.SMA`, `talib.RSI`, ...) — takes/returns NumPy arrays.
  * **Abstract API** (`talib.abstract.Function('rsi')` or `import talib.abstract as ta`) — works directly with pandas/dict-like OHLCV input and is commonly used inside frameworks like freqtrade.

## Repository Structure & Key Directories
* `/talib/` - Python package
  * `/talib/func.py` (generated) - Function-style indicator wrappers
  * `/talib/abstract.py` - Abstract API for dict/DataFrame-based OHLCV input
  * `/talib/stream.py` - Streaming/incremental calculation helpers (latest value only)
* `/docs/` - Function group reference (Overlap Studies, Momentum, Volatility, Volume, Cycle, Pattern Recognition, Statistic Functions)
* Tests demonstrate expected input shapes (`open`, `high`, `low`, `close`, `volume` as NumPy float64 arrays)

## Standard Operational Commands
* **Install C library first** (Linux example): build/install `ta-lib` C library from source or via system package manager, then:
* **Install Python wrapper**: `pip install TA-Lib`
* Some platforms now offer prebuilt wheels (`pip install ta-lib-binary` style alternatives) — check current PyPI status if build errors occur.

## Standard Python API Usage
```python
import talib
import numpy as np

close = np.random.random(100)

# Function API
sma = talib.SMA(close, timeperiod=20)
rsi = talib.RSI(close, timeperiod=14)
macd, macdsignal, macdhist = talib.MACD(close, fastperiod=12, slowperiod=26, signalperiod=9)

# Abstract API (DataFrame-friendly)
import talib.abstract as ta
import pandas as pd
df = pd.DataFrame({"open": ..., "high": ..., "low": ..., "close": ..., "volume": ...})
df["rsi"] = ta.RSI(df, timeperiod=14)
```

## AI Assistant Guidelines for this Project
1. **Input format**: always require `open/high/low/close/volume` as `float64` NumPy arrays (or pandas Series of float64) — integer arrays or NaNs commonly cause errors.
2. **Function vs Abstract**: use `talib.abstract` when working with pandas DataFrames (e.g. inside freqtrade strategies); use the plain function API for raw NumPy array workflows.
3. **Installation issues**: if a user reports `pip install TA-Lib` failing, the root cause is almost always the missing C library — guide them to install the C library/binary wheel for their OS first.
4. **Indicator parameters**: when suggesting an indicator, include sensible default `timeperiod`/parameter values matching common usage (e.g. RSI 14, MACD 12/26/9, BBANDS 20/2).
5. **Pattern recognition functions**: TA-Lib's candlestick pattern functions (`CDL*`) return integer signals (100/-100/0) — explain this when used.
