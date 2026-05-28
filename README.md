# Synthetic Index Indicator Alert — MQL4 Script

A MetaTrader 4 script that constructs a **composite synthetic index** by averaging the current closing prices of three configurable market instruments via `iClose()`, tracks the cycle-to-cycle change in the composite value against a persistent `previousIndex` global, and fires upward or downward shift alerts when the absolute change between consecutive cycles equals or exceeds the configurable `AlertThreshold` — providing a multi-instrument composite momentum signal in a single unified alert stream.

---

## Overview

Monitoring a single market instrument in isolation misses the broader context of correlated market behavior. Major equity indices — the S&P 500, Dow Jones, and NASDAQ — move with high correlation during risk-on and risk-off regime shifts. By constructing a synthetic composite of all three, this script captures regime transitions more robustly than any single index: a simultaneous move across all three instruments produces a composite shift three times larger than a move in one instrument alone, making the synthetic index more sensitive to coordinated market movements and less sensitive to single-instrument idiosyncratic noise. The equal-weight averaging approach — `sum / 3.0` of the three closing prices — gives each index identical influence on the composite, avoiding the market-cap weighting biases present in the actual index construction. This makes the composite a pure price-movement signal rather than a capitalization-weighted measure.

---

## Features

- **Three-instrument equal-weight composite** — `CalculateSyntheticIndex()` computes `sum = iClose(idx1) + iClose(idx2) + iClose(idx3)`; validates `MathMin(iBars(idx1), MathMin(iBars(idx2), iBars(idx3))) >= 1` before fetching; returns `sum / 3.0`
- **Absolute cycle-to-cycle shift detection** — `MathAbs(syntheticIndex − previousIndex) >= AlertThreshold` evaluates the magnitude of the composite's move between consecutive minute cycles, independent of direction
- **Directional classification** — `syntheticIndex > previousIndex` → **Upward Shift Detected**; `syntheticIndex < previousIndex` → **Downward Shift Detected** — applied only after the threshold gate passes
- **`previousIndex` persistent state** — global double updated unconditionally each cycle regardless of alert state; ensures the delta is always measured from the true prior reading
- **Rich alert message** — `AlertSyntheticIndex()` formats with both current and previous composite values via `"Current Value: %.2f\nPrevious Value: %.2f"` for full shift context
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateSyntheticIndex()` fetches the current close for all three instruments and returns their average
2. `MathAbs(syntheticIndex − previousIndex) >= AlertThreshold` evaluated; directional classification applied
3. `previousIndex = syntheticIndex` updated unconditionally at cycle end

---

## Input Parameters

| Parameter        | Type            | Default     | Description                                                         |
|------------------|-----------------|-------------|---------------------------------------------------------------------|
| `Index1`         | string          | `US500`     | First index symbol                                                  |
| `Index2`         | string          | `US30`      | Second index symbol                                                 |
| `Index3`         | string          | `NASDAQ`    | Third index symbol                                                  |
| `Timeframe`      | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for composite calculation                                 |
| `LookbackPeriod` | int             | `14`        | Reserved for future MA-based smoothing extension                    |
| `AlertThreshold` | double          | `2.0`       | Minimum absolute composite shift between cycles to trigger an alert |
| `EnableAlerts`   | bool            | `true`      | Fire an on-screen/sound alert                                       |
| `EnableEmail`    | bool            | `false`     | Send an email notification                                          |
| `EnablePush`     | bool            | `false`     | Send a mobile push notification                                     |

---

## Alert Message Format

```
Upward Shift Detected detected in Synthetic Index (Timeframe: PERIOD_H1)
Current Value: 15423.67
Previous Value: 15419.20
```

---

## Installation

1. Copy `Synthetic_Index_Indicator_001.mq4` to `MQL4/Scripts/`
2. Compile in MetaEditor (F7)
3. Ensure all three index symbols are available in Market Watch
4. Drag onto any chart from Navigator → Scripts; configure inputs and click **OK**

> **Note:** Symbol names must match exactly as they appear in your broker's Market Watch (e.g. `US500`, `US30`, `NASDAQ`).

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)
- All three index symbols available and streaming in Market Watch

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
