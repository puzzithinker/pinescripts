# AIO Strategy Builder — Complete Usage Guide

## Table of Contents

1. [Overview](#1-overview)
2. [Core Concepts](#2-core-concepts)
3. [Signal Flow Architecture](#3-signal-flow-architecture)
4. [Setup & Basic Configuration](#4-setup--basic-configuration)
5. [Leading Indicator (Signal Generator)](#5-leading-indicator-signal-generator)
6. [Confirmation Filters (Gate System)](#6-confirmation-filters-gate-system)
7. [Switch Board (Visual Overlays)](#7-switch-board-visual-overlays)
8. [Signal Processing](#8-signal-processing)
9. [Presets](#9-presets)
10. [Dashboard](#10-dashboard)
11. [Alerts](#11-alerts)
12. [Indicator Deep Dive](#12-indicator-deep-dive)
13. [Strategy Recipes](#13-strategy-recipes)
14. [Troubleshooting & FAQ](#14-troubleshooting--faq)
15. [Pine Script Limitations](#15-pine-script-limitations)

---

## 1. Overview

The **AIO Strategy Builder** is a comprehensive TradingView indicator that combines 40+ technical indicators into a single, configurable signal generation system. Instead of running dozens of separate indicators, you use one script with a unified signal flow:

1. **Pick one leading indicator** — this generates the raw buy/sell signal
2. **Enable confirmation filters** — these must all agree before a signal is shown
3. **Configure overlays** — visual elements drawn on your chart

The result is a highly customizable trading signal system where you control exactly which indicators participate and how they interact.

**Key idea**: Most trading systems fail because they use too few confirmation signals (high false positive rate) or too many (never triggers). The AIO lets you find the right balance by mixing and matching indicators as filters.

---

## 2. Core Concepts

### Leading Indicator vs Confirmation Filter

This is the most important distinction in the AIO:

| Concept | Role | Behavior | Example |
|---------|------|----------|---------|
| **Leading Indicator** | Signal *generator* | Fires a **single-bar event** — "something just happened!" | Range Filter direction change, SMC trend change, MACD crossover |
| **Confirmation Filter** | Signal *gate* | Provides a **persistent state** — "conditions are right" or "conditions are wrong" | EMA filter (price above 200 EMA = bullish state), RSI above 50, SMC swing trend is bullish |

Think of it like a security checkpoint:
- The **leading indicator** is the person approaching the gate (the event)
- Each **confirmation filter** is a guard that checks one condition
- **All guards must approve** (in AND-chain mode) for the person to pass through

### Event-Based vs State-Based Signals

| Type | Description | Analogy |
|------|-------------|---------|
| **Event-based** | True for exactly one bar when something changes, then goes back to false | A doorbell — rings once when pressed |
| **State-based** | True for every bar while a condition holds | A light switch — stays on until turned off |

Leading indicators are event-based (they fire once when conditions change). Confirmation filters are state-based (they stay true as long as the condition holds). This combination prevents signal spam — the leading indicator ensures signals only fire at meaningful moments, while confirmation filters ensure the broader context supports the trade.

### Signal Expiry

Signals have a shelf life. By default, if all confirmation filters don't align within **3 bars** of the leading indicator firing, the signal expires and no trade is shown. This prevents stale signals from executing when market conditions have already changed.

### Alternate Signal Mode

When enabled (default), the system alternates between long and short signals. After a long signal, the next signal must be short, and vice versa. This prevents duplicate same-direction signals and forces the system to wait for a genuine reversal.

---

## 3. Signal Flow Architecture

```
┌─────────────────────────────────┐
│   LEADING INDICATOR (1 of 42)   │  ← Generates raw signal event
│   e.g., Range Filter flips      │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│   CONFIRMATION FILTERS (0-38)   │  ← All enabled filters must pass
│   e.g., EMA above 200 ✓        │
│        RSI above 50 ✓          │
│        MACD bullish ✓          │
│        Volume above MA ✗ → BLOCK│
└──────────────┬──────────────────┘
               │ (only if all pass)
               ▼
┌─────────────────────────────────┐
│   SIGNAL EXPIRY (default: 3)   │  ← Must align within N bars
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│   ALTERNATE SIGNAL MODE         │  ← No consecutive same-direction
└──────────────┬──────────────────┘
               │
               ▼
        LONG / SHORT SIGNAL
        (chart label + alert)
```

**Alternative mode — Signal Scoring**: Instead of requiring ALL filters to pass, the scoring mode counts how many enabled filters pass and triggers a signal when the count reaches a configurable threshold (e.g., "at least 5 of 10 enabled filters pass"). This is more flexible and generates more signals.

---

## 4. Setup & Basic Configuration

When you first add the indicator to your chart, you'll see the **Indicator Setup** section at the top of the settings panel:

### Indicator Setup Inputs

| Setting | Default | What It Does |
|---------|---------|-------------|
| **Signal Expiry Candle Count** | 3 | How many bars after a leading signal for confirmations to align. Higher = more signals but potentially stale. Lower = fewer but fresher signals. |
| **Alternate Signal** | On | Prevents consecutive same-direction signals. Recommended to keep on. |
| **Signal Scoring Mode** | Off | Switches from AND-chain (all must pass) to scoring (N must pass). |
| **Min Score** | 3 | When scoring is on, minimum number of passing filters needed. |
| **Show Long/Short Signal** | On | Shows "Long"/"Short" labels on the chart. Turn off for cleaner charts. |
| **Preset** | Custom | Quick configuration presets (see [Presets](#9-presets)). |
| **Show Dashboard** | On | Shows confirmation filter status table on chart. |
| **Show Indicator Status** | Off | Shows detailed indicator state table (SMC, BC, Vix, Squeeze, EMA, STC). |
| **Dashboard Position** | Bottom Right | Position and size of the dashboard table. |

### Quick Start: Minimal Setup

1. Set **Leading Indicator** to "Range Filter" (default)
2. Enable **1-3 confirmation filters** you understand (e.g., EMA Filter, MACD, RSI)
3. Leave everything else at defaults
4. Watch the dashboard to see which filters pass/block signals

---

## 5. Leading Indicator (Signal Generator)

The leading indicator is the **trigger** — it decides *when* to look for a trade. Only one can be active at a time. Choose based on your trading style:

### Trend-Following Leading Indicators

These generate signals when a trend is confirmed to be starting or continuing:

| Indicator | What It Detects | Best For |
|-----------|----------------|----------|
| **Range Filter** | Price breaks out of a dynamic range | General purpose, default choice |
| **Supertrend** | Trend direction changes via ATR-based line | Clean trend markets |
| **Half Trend** | Trend changes with buy/sell arrows | Swing trading |
| **Ichimoku Cloud** | Price crosses the cloud or tenkan/kijun cross | Forex, longer timeframes |
| **MACD** | MACD line crosses signal line or zero | Classic trend confirmation |
| **Hull Suite** | Hull MA direction changes | Fast trend detection |
| **Rational Quadratic Kernel (RQK)** | Nadaraya-Watson estimator crossover | Smooth trend signals |

### Momentum Leading Indicators

These generate signals based on momentum shifts:

| Indicator | What It Detects | Best For |
|-----------|----------------|----------|
| **RSI** | RSI crosses signal line or levels | Overbought/oversold reversals |
| **Stochastic** | %K/%D crossover in OB/OS zones | Range-bound markets |
| **CCI** | CCI crosses zero or threshold | Cycle identification |
| **True Strength Indicator (TSI)** | TSI signal line cross | Smoothed momentum |
| **Schaff Trend Cycle (STC)** | STC cycle crossover | Fast cycle trading |
| **Awesome Oscillator** | AO zero line or saucer cross | Elliott wave traders |

### Oscillation / Mean-Reversion Leading Indicators

| Indicator | What It Detects | Best For |
|-----------|----------------|----------|
| **B-Xtrender** | Short/long term trend alignment | Medium-term reversals |
| **Waddah Attar Explosion** | MACD + BB expansion | Volatility breakout |
| **QQE Mod** | Quantitative Qualitative Estimation cross | Fast momentum shifts |
| **Williams Vix Fix** | Market bottom detection (elevated VixFix crosses down) | Panic selling reversals |
| **Squeeze Momentum** | Bollinger Band squeeze release | Breakout anticipation |

### Smart Money Leading Indicators

| Indicator | What It Detects | Best For |
|-----------|----------------|----------|
| **Smart Money Concepts** | SMC swing trend change (BOS/CHoCH) | Institutional trading style |
| **Breakout Channels** | Price breaks out of consolidation channel | Breakout trading |

### Composite Leading Indicators

| Indicator | What It Detects | Best For |
|-----------|----------------|----------|
| **Trend Meter** | Multiple trend meters align in same direction | Multi-factor trend confirmation |
| **2 EMA Cross** | Fast EMA crosses slow EMA | Simple trend following |
| **3 EMA Cross** | Three EMAs align in order | Stronger trend confirmation |
| **Bull Bear Power Trend** | Elder Ray bull/bear power | Trend strength assessment |

### Other Leading Indicators

| Indicator | What It Detects |
|-----------|----------------|
| **Trend Direction Force Index (TDFI)** | Force of current trend direction |
| **Trendline Breakout** | Price breaks drawn trendlines |
| **Range Detector** | Price breaks out of detected range |
| **Heiken-Ashi Candlestick Oscillator** | HA oscillator crosses zero |
| **Donchian Trend Ribbon** | Donchian channel trend direction |
| **Rate of Change (ROC)** | Price rate of change crosses zero |
| **VWAP** | Price crosses VWAP |
| **Detrended Price Oscillator (DPO)** | DPO crosses zero |
| **BB Oscillator** | BB oscillator enters/exits bands |
| **Chandelier Exit** | Chandelier exit direction change |
| **DMI (ADX)** | ADX trend strength threshold |
| **Parabolic SAR (PSAR)** | SAR flips direction |
| **SSL Channel** | SSL buy/sell channel switch |
| **Chaikin Money Flow** | CMF crosses zero |
| **Vortex Index** | VI+ crosses VI- |
| **Volatility Oscillator** | Volatility state change |
| **Wolfpack Id** | Wolfpack trend alignment |

---

## 6. Confirmation Filters (Gate System)

Confirmation filters are the **quality control** layer. After the leading indicator fires, every enabled filter must approve the signal for it to appear. You can enable zero filters (all signals pass) or stack many (very strict, fewer signals).

### How Confirmation Filters Work

Each filter outputs a **long** and **short** boolean state every bar:

- `true` = "I approve this direction"
- `false` = "I block this direction"

When all enabled filters say `true` for the same direction within the signal expiry window, a trade signal is generated.

### Filter Categories

#### Trend Direction Filters
These confirm that price is moving in the right direction:

| Filter | What It Checks | Long Passes When | Short Passes When |
|--------|---------------|-----------------|------------------|
| **EMA Filter** | Price vs single EMA | Price above EMA 200 | Price below EMA 200 |
| **2 EMA Cross** | Fast EMA vs slow EMA | Fast above slow | Fast below slow |
| **3 EMA Cross** | Three EMAs in order | EMA9 > 21 > 55 | EMA9 < 21 < 55 |
| **SuperTrend** | SuperTrend direction | ST is bullish | ST is bearish |
| **Half Trend** | HalfTrend arrow direction | HT buy arrow active | HT sell arrow active |
| **MACD** | MACD vs signal or zero | MACD bullish crossover | MACD bearish crossover |
| **Hull Suite** | Hull MA direction | Hull is bullish | Hull is bearish |
| **McGinley Dynamic** | McGinley MA direction | Price above McGinley | Price below McGinley |
| **Parabolic SAR** | SAR position | SAR below price | SAR above price |
| **Ichimoku Cloud** | Full ichimoku conditions | Above cloud + tenkan>kijun + green cloud + lagging above | Opposite |
| **SSL Channel** | SSL buy/sell | SSL in buy mode | SSL in sell mode |

#### Momentum Filters
These confirm that momentum supports the trade:

| Filter | What It Checks | Key Concept |
|--------|---------------|-------------|
| **RSI** | RSI level or crossover | RSI above/below signal line or threshold |
| **RSI MA Direction** | Direction of RSI's moving average | RSI MA is rising/falling |
| **RSI Limit** | RSI within a band | RSI between upper/lower limits |
| **RSI MA Limit** | RSI MA within a band | RSI MA between limits |
| **Stochastic** | %K/%D crossover | Stoch crossover in/out of OB/OS zones |
| **CCI** | CCI zero line or threshold | CCI above/below zero |
| **True Strength Indicator** | TSI signal cross | TSI direction confirms trade |
| **Awesome Oscillator** | AO zero line or saucer | AO confirms momentum direction |
| **Schaff Trend Cycle** | STC cycle position | STC above/below trigger |
| **B-Xtrender** | Short/long term trend alignment | Both timeframes agree |
| **Bull Bear Power Trend** | Elder Ray power | Bull/bear power confirms direction |
| **BB Oscillator** | BB oscillator bands | Entering/exiting upper/lower bands |
| **Detrended Price Oscillator** | DPO zero line | DPO above/below zero |
| **Rate of Change** | ROC zero line | ROC above/below zero |
| **QQE Mod** | QQE line/bar crossover | QQE confirms momentum shift |

#### Volatility & Range Filters
These filter based on whether the market is trending or ranging:

| Filter | What It Checks | Key Concept |
|--------|---------------|-------------|
| **Range Filter** | Dynamic range breakout | Price is outside the calculated range |
| **Rational Quadratic Kernel** | NW estimator crossover | Smooth trend direction confirmed |
| **Donchian Trend Ribbon** | Donchian channel direction | Ribbon color confirms trend |
| **Range Detector** | Detected price range | Price broke out of range |
| **Chandelier Exit** | ATR-based trailing stop | Chandelier confirms trend direction |
| **Choppiness Index** | Is the market trending? | CI below threshold = trending (allows trades) |
| **Damiani Volatmeter** | Is volatility trending? | DV confirms trending conditions |
| **Volatility Oscillator** | Volatility state | VO confirms volatility direction |
| **Squeeze Not Active** | BB inside KC? | Market is NOT in squeeze = trending |

#### Volume & Flow Filters
These confirm that volume supports the trade:

| Filter | What It Checks | Key Concept |
|--------|---------------|-------------|
| **Volume** | Volume vs moving average | Volume above MA confirms participation |
| **Chaikin Money Flow** | CMF direction | Money flowing in/out |
| **VWAP** | Price vs VWAP | Institutional price reference |
| **Waddah Attar Explosion** | MACD + BB expansion | Volatility + momentum alignment |
| **Vortex Index** | VI+ vs VI- | Trend direction via vortex |

#### Smart Money Filters
These use institutional trading concepts:

| Filter | What It Checks | Key Concept |
|--------|---------------|-------------|
| **Smart Money Concepts** | SMC swing trend bias | Bullish swing = longs allowed, bearish = shorts allowed |
| **SMC HTF Trend** | Higher-timeframe SMC bias | HTF swing trend confirms direction (e.g., 4H trend is bullish) |
| **OB Mitigation** | Order block price breach | Bearish OB mitigated = bullish signal, bullish OB mitigated = bearish signal |
| **Breakout Channels** | Channel breakout direction | Bullish breakout confirms long, bearish confirms short |
| **FVG-OB Confluence** | FVG overlaps with Order Block | Both are bullish = strong support zone, both bearish = strong resistance |
| **Williams Vix Fix** | Market panic state | Not in panic = safe to long; in panic = potential short/bottom |
| **Squeeze Momentum** | Momentum direction after squeeze | Positive momentum = bullish bias, negative = bearish bias |

#### Session Filter

| Filter | What It Checks | Key Concept |
|--------|---------------|-------------|
| **Session Filter** | Time of day | Only allows signals during specified trading session (e.g., 0900-1600 for London+NY) |

#### Composite Filter

| Filter | What It Checks | Key Concept |
|--------|---------------|-------------|
| **Trend Meter** | Multiple meters/bars aligned | All 3 trend meters (and optionally 2 trend bars + wavetrend) agree on direction |

### Trend Filters vs Is-Trending Filters

Two special filters deserve explanation:

- **Choppiness Index (`respectci`)**: Uses the same condition (`ci_filter`) for both long and short. This is because CI measures *whether* the market is trending, not *which direction*. When CI is below the threshold, the market is trending — so both longs and shorts are allowed.
- **Damiani Volatmeter (`respectdv`)**: Uses `dvup` for both long and short. Same reason — it measures *is there a trend*, not the trend direction.

These are "gate" filters — they don't have a directional opinion. They either allow all signals (market is trending) or block all signals (market is ranging/choppy).

---

## 7. Switch Board (Visual Overlays)

The Switch Board controls **what gets drawn on your chart**. These are purely visual — turning them off doesn't affect signal generation (that's what confirmation filters do).

| Switch | What It Draws | Notes |
|--------|--------------|-------|
| **EMA** | 5 configurable moving average lines | Also shows crossover markers if EMA Cross Signals is on |
| **Supply/Demand Zone** | Supply and demand zones with labels | On by default |
| **PSAR** | Parabolic SAR dots | |
| **Ichimoku Cloud** | Full ichimoku overlay (cloud, lines) | |
| **Heiken-Ashi Candles** | Heiken-Ashi candlestick overlay | Replaces regular candles |
| **Fibonacci** | Auto-drawn Fibonacci retracement levels | |
| **Range Detector** | Range zone visualization | |
| **VWAP** | VWAP line | |
| **Bollinger Band** | Bollinger Bands overlay | |
| **Supertrend** | SuperTrend line | |
| **Half Trend** | HalfTrend arrows | |
| **Range Filter** | Range Filter visualization | |
| **STC** | STC circle markers + background coloring | Auto-enables when STC is leading indicator |
| **PVSRA** | Price Volume Support Resistance Analysis | On by default |
| **Liquidity Zone** | Liquidity zones visualization | |
| **Fair Value Gap (FVG)** | Fair value gap boxes | |
| **Pivot Levels** | Pivot point levels | |
| **Fractal** | Fractal markers | |
| **Market Sessions** | London/NY/Tokyo/HK/Sydney session highlights | On by default |
| **Smart Money Concepts** | SMC structure labels, order blocks, FVGs, strong/weak H/L | On by default; auto-enables when SMC is leading |
| **Breakout Channels** | Consolidation channel boxes | Auto-enables when BC is leading |
| **Squeeze Dots** | Squeeze state dots below bars | Auto-enables when Squeeze is leading |

**Auto-enable behavior**: When you select certain indicators as the leading indicator, their overlay automatically turns on even if the switch is off. This ensures you can see the indicator's signals on your chart. This applies to: STC, Squeeze, SMC, and Breakout Channels.

---

## 8. Signal Processing

### Signal Expiry

After the leading indicator fires, the system starts a countdown. By default, you have **3 bars** for all confirmation filters to align. If they don't align within that window, the signal is discarded.

- **Higher expiry** (5-10): More signals, but some may be stale. Good for slow-moving markets or higher timeframes.
- **Lower expiry** (1-2): Fewer signals, but more timely. Good for scalping or fast markets.

### Alternate Signal Mode

When **on** (default): After a long signal, the system won't generate another long until a short signal has occurred first. This prevents signal spam in strong trends.

When **off**: The same direction can signal repeatedly. Useful when you want to pyramid into positions.

### Signal Scoring Mode

Instead of requiring **all** confirmation filters to pass, scoring mode counts how many pass and triggers when the count reaches a threshold:

| Setting | Behavior |
|---------|----------|
| **Signal Scoring Mode: Off** | All enabled filters must pass (AND-chain). Strict, fewer signals. |
| **Signal Scoring Mode: On** | At least N enabled filters must pass. Flexible, more signals. |
| **Min Score: 3** | At least 3 of your enabled filters must agree. |
| **Min Score: 5** | At least 5 of your enabled filters must agree. |
| **Min Score: 1** | Any single enabled filter agreeing is enough. Very loose. |

**When to use scoring mode**: When you have many confirmation filters enabled (5+) and the AND-chain is too strict (rarely generates signals). Scoring lets you find the middle ground between "all must pass" and "any can pass."

The dashboard shows your current score when scoring mode is active.

---

## 9. Presets

Presets override your individual filter settings with pre-configured combinations. The preset dropdown is in the Indicator Setup section.

| Preset | What It Does | Best For |
|--------|-------------|----------|
| **Custom** | Uses your individual settings — no overrides | Full control, experienced users |
| **SMC Scalper** | Enables: SMC, SMC HTF Trend, OB Mitigation, FVG-OB Confluence | Intraday trading with smart money concepts |
| **Trend Follower** | Enables: EMA Filter, 2 EMA Cross, SuperTrend, MACD | Trend-following on higher timeframes |
| **Breakout Trader** | Enables: Range Filter, Breakout Channels, Squeeze Not Active | Breakout strategies in trending markets |
| **All Filters** | Enables every single confirmation filter | Maximum strictness, very few but high-quality signals |

**How presets work**: When you select a preset, it forces certain `respect*` toggles to `true` regardless of your individual settings. It does NOT disable any toggles you've already enabled — it only adds more. Your individual toggle settings are preserved and will take effect when you switch back to "Custom."

---

## 10. Dashboard

### Main Dashboard

Shows the status of every enabled confirmation filter in real-time:

| Column | Content |
|--------|---------|
| **Left** | Filter name |
| **Center** | Long status (checkmark or X) |
| **Right** | Short status (checkmark or X) |

A green checkmark means the filter currently supports that direction. A red X means it blocks it. For a signal to fire, all rows must show green checkmarks in the same column.

### Indicator Status Dashboard

Enable **"Show Indicator Status"** to see a second table showing the current state of key indicators:

| Row | Shows |
|-----|-------|
| **SMC Trend** | Bullish / Bearish / Neutral |
| **Breakout Ch.** | Bull Breakout / Bear Breakout / In Channel / None |
| **Vix Fix** | Normal / Elevated |
| **Squeeze** | Squeezing / Released Bull / Released Bear |
| **EMA** | Above / Below (relative to primary MA) |
| **STC** | Overbought / Oversold / Neutral |

---

## 11. Alerts

The AIO provides 18 alert conditions you can use with TradingView's alert system:

### Core Signal Alerts
| Alert | Triggers When |
|-------|--------------|
| **Buy Alert** | A confirmed long signal is generated |
| **Sell Alert** | A confirmed short signal is generated |
| **Buy or Sell Alert** | Either a long or short signal is generated |

### Smart Money Alerts
| Alert | Triggers When |
|-------|--------------|
| **SMC Bullish BOS** | Swing bullish Break of Structure detected |
| **SMC Bullish CHoCH** | Swing bullish Change of Character detected |
| **SMC Bearish BOS** | Swing bearish Break of Structure detected |
| **SMC Bearish CHoCH** | Swing bearish Change of Character detected |
| **SMC Equal Highs** | Equal highs detected (liquidity pool) |
| **SMC Equal Lows** | Equal lows detected (liquidity pool) |
| **SMC HTF Trend Bullish** | Higher-timeframe SMC trend turns bullish |
| **SMC HTF Trend Bearish** | Higher-timeframe SMC trend turns bearish |
| **SMC Bearish OB Mitigated** | Bearish order block broken — bullish signal |
| **SMC Bullish OB Mitigated** | Bullish order block broken — bearish signal |
| **FVG-OB Bullish Confluence** | Price enters a bullish FVG-OB overlap zone |
| **FVG-OB Bearish Confluence** | Price enters a bearish FVG-OB overlap zone |

### Other Indicator Alerts
| Alert | Triggers When |
|-------|--------------|
| **BC Bullish Breakout** | Price breaks above a breakout channel |
| **BC Bearish Breakout** | Price breaks below a breakout channel |
| **BC New Channel** | A new consolidation channel forms |
| **Vix Fix Bottom Signal** | Potential market bottom detected (elevated VixFix crosses down) |
| **Vix Fix Exhaustion Signal** | VixFix drops from elevated level |
| **Squeeze Release Bull** | Squeeze ends with bullish momentum |
| **Squeeze Release Bear** | Squeeze ends with bearish momentum |
| **EMA Cross Up** | Fast EMA crosses above slow EMA |
| **EMA Cross Down** | Fast EMA crosses below slow EMA |
| **Trend Meter Bullish Align** | All trend meters align bullish |
| **Trend Meter Bearish Align** | All trend meters align bearish |

### Setting Up Alerts

1. Right-click on the chart → **Add Alert**
2. Select **AIO Strategy Builder** from the condition dropdown
3. Choose the specific alert condition from the sub-dropdown
4. Configure notification method (popup, email, phone, webhook)

---

## 12. Indicator Deep Dive

### Smart Money Concepts (SMC)

SMC is a trading methodology based on institutional order flow. The AIO implements key SMC concepts:

| Concept | Abbreviation | What It Means |
|---------|-------------|---------------|
| **Break of Structure** | BOS | Price breaks a previous swing high/low in the trend direction, confirming trend continuation |
| **Change of Character** | CHoCH | Price breaks a swing in the opposite direction, signaling a potential trend reversal |
| **Order Block** | OB | The last opposing candle before a strong move — institutional entry zone |
| **Fair Value Gap** | FVG | A gap between candles where price moved too fast — likely to be revisited (imbalance) |
| **Strong/Weak High/Low** | — | Strong levels have unmitigated order blocks behind them; weak levels have been breached |
| **Equal Highs/Lows** | EQH/EQL | Multiple swing points at the same level — liquidity pool likely to be targeted |

**SMC Settings** (in the SMC input group):
- **Mode**: "Historical" shows all structure labels; "Present" shows only the latest
- **Swing Length**: How many bars to look back for swing detection. Higher = fewer but more significant swings
- **Show Order Blocks**: Toggle OB visualization
- **Show FVGs**: Toggle FVG visualization
- **Style**: "Colored" or "Monochrome"

**SMC as Leading Indicator**: Fires on trend change events (BOS/CHoCH). After the first bar of a new trend, the leading signal returns to false until the next trend change. The confirmation filter (`respectsmc`) stays true as long as the swing trend direction supports your trade.

**SMC HTF Trend**: Uses `request.security` to read the SMC swing trend from a higher timeframe. For example, if you're on 15m and set HTF to "240" (4H), the filter only allows longs when the 4H SMC trend is bullish. This adds multi-timeframe confluence.

### Breakout Channels

Based on AlgoAlpha's Smart Money Breakout Channels. Detects consolidation periods where price moves sideways and draws channel boxes. When price breaks out of a channel, it generates a signal.

| Setting | What It Does |
|---------|-------------|
| **Nested Channels** | Allow overlapping channels (more complex) or single channel only (cleaner) |
| **Strong Closes Only** | Require >50% of candle body outside channel for breakout (reduces wick false signals) |
| **Box Detection Length** | Bars to detect channel formation. Lower = more channels, higher = fewer but more significant |
| **Max Channels** | Capped at 30 to conserve box resources |

**As Leading Indicator**: Bullish/bearish breakout is a single-bar event. The signal disappears after the breakout bar.

**As Confirmation Filter**: Confirms direction based on whether the last breakout was bullish or bearish.

### Williams Vix Fix

Detects market panic/bottom conditions by measuring how far the close is below the lowest low over a lookback period. When the Vix Fix is elevated, it means the market is in a panic (potential bottom). When it drops back down, the panic may be exhausting.

| Signal | Behavior |
|--------|----------|
| **Leading (vix_long)** | Fires when VixFix first crosses above threshold — potential bottom |
| **Leading (vix_short)** | Fires when VixFix drops below threshold — exhaustion of panic |
| **Confirmation (vix_conf_long)** | True when VixFix is NOT elevated — safe to long (no panic) |
| **Confirmation (vix_conf_short)** | True when VixFix IS elevated — caution, but potential bottom |

### Squeeze Momentum

Based on LazyBear's indicator. Detects when Bollinger Bands are inside Keltner Channels (squeeze = low volatility, breakout coming). When the squeeze releases, momentum direction indicates the likely breakout direction.

| Signal | Behavior |
|--------|----------|
| **Leading (sqz_long/short)** | Fires on squeeze release with positive/negative momentum |
| **Confirmation (sqz_conf_long/short)** | True based on momentum direction — works every bar, not just on release |
| **Squeeze Not Active filter** | True when market is NOT squeezing (i.e., is trending) — acts as an is-trending gate |

**Visual**: When the Squeeze Dots switch is on, you'll see orange dots during squeeze (anticipation) and green/red dots on release (direction).

### Trend Meter

A composite indicator that combines multiple trend assessment methods into a single signal. It uses:

- **3 Trend Meters**: Each can be set to one of 8 methods (MACD crossover variants, Mom/Dad Cross, RSI variants, Trend Candles, N/A)
- **2 Trend Bars**: Each can be set to MA Crossover, MA Direction (fast/slow), or N/A
- **3 Alignment Modes**:

| Mode | Signal Condition |
|------|-----------------|
| **3 TM Align** | All 3 trend meters show the same direction |
| **3 TM + 2 TB Align** | All 3 meters + both trend bars aligned |
| **3 TM + 2 TB + WT** | Above + wavetrend crossover confirms |

**As Leading Indicator**: Fires when alignment changes from bearish to bullish (or vice versa) — event-based.

**As Confirmation Filter**: True as long as the meters/bars remain aligned in one direction — state-based.

### FVG-OB Confluence

When a Fair Value Gap spatially overlaps with an Order Block, it creates a high-probability zone where two types of institutional interest coincide:

- **Bullish confluence**: Bullish FVG overlaps with bullish OB → strong support zone
- **Bearish confluence**: Bearish FVG overlaps with bearish OB → strong resistance zone

The confirmation filter (`respectfv_ob`) confirms long when a bullish confluence zone exists, and confirms short when a bearish one exists.

### EMA Cross Display

When the EMA overlay is enabled (`switch_ema`), the AIO can show crossover markers on the chart:

- **Arrow up** (below bar): Fast MA crosses above slow MA → bullish crossover
- **Arrow down** (above bar): Fast MA crosses below slow MA → bearish crossover

Controlled by the **"EMA Cross Signals"** toggle in the Switch Board. Alert conditions are available for both directions.

### STC Zone Background

When the STC overlay is enabled (`switch_stc`), the AIO colors the chart background based on the Schaff Trend Cycle's position:

| Background Color | STC Position | Meaning |
|-----------------|-------------|---------|
| **Green** | Above upper band | Strong bullish momentum |
| **Red** | Below lower band | Strong bearish momentum |
| **Orange** | Between bands | Neutral / transitioning |

The transparency is controlled by **"Background Transparency"** in the STC settings (default 90 = nearly invisible; lower = more visible).

---

## 13. Strategy Recipes

Here are proven configurations for common trading styles:

### Recipe 1: SMC Scalper (Intraday)

```
Preset: SMC Scalper
Leading Indicator: Smart Money Concepts
Confirmation Filters (enabled by preset):
  ✓ Smart Money Concepts
  ✓ SMC HTF Trend (set to "60" for 1H on 5m chart)
  ✓ OB Mitigation
  ✓ FVG-OB Confluence
Additional toggles:
  Switch Board: Smart Money Concepts ✓
  Signal Expiry: 3
  Alternate Signal: On
```

**Logic**: SMC leading fires on BOS/CHoCH events. The SMC confirmation ensures swing trend agrees. HTF filter ensures you're trading with the higher timeframe trend. OB Mitigation and FVG-OB add confluence when institutional zones are triggered.

### Recipe 2: Trend Follower (Swing)

```
Preset: Trend Follower
Leading Indicator: Range Filter or Supertrend
Confirmation Filters (enabled by preset):
  ✓ EMA Filter (200)
  ✓ 2 EMA Cross (50/200)
  ✓ SuperTrend
  ✓ MACD
Additional toggles:
  Switch Board: EMA ✓, Supertrend ✓
  Signal Expiry: 5
  Alternate Signal: On
```

**Logic**: Range Filter/Supertrend detects trend starts. EMA 200 ensures you're on the right side of the long-term trend. 2 EMA Cross adds medium-term trend confirmation. MACD adds momentum confirmation. Signal expiry of 5 gives more time for alignment.

### Recipe 3: Breakout Trader

```
Preset: Breakout Trader
Leading Indicator: Breakout Channels or Squeeze Momentum
Confirmation Filters (enabled by preset):
  ✓ Range Filter
  ✓ Breakout Channels
  ✓ Squeeze Not Active
Additional toggles:
  Switch Board: Breakout Channels ✓, Squeeze Dots ✓
  Signal Expiry: 3
  Alternate Signal: On
```

**Logic**: Breakout Channels or Squeeze Momentum detects the breakout moment. Range Filter confirms the trend direction. Squeeze Not Active ensures the market is actually trending (not still consolidating). This combination only fires when a genuine breakout occurs with trend confirmation.

### Recipe 4: Conservative All-Clear

```
Preset: All Filters
Leading Indicator: Range Filter
Signal Expiry: 5
Signal Scoring Mode: On
Min Score: 5
Alternate Signal: On
```

**Logic**: All filters enabled, but scoring mode requires only 5 to pass instead of all 38. This gives high-quality signals that have broad indicator support without requiring every single indicator to agree. The scoring threshold of 5 means at least 5 different indicator categories confirm the trade.

### Recipe 5: Mean Reversion (Range-Bound Markets)

```
Preset: Custom
Leading Indicator: Williams Vix Fix or Stochastic
Confirmation Filters:
  ✓ RSI (RSI level mode, overbought/oversold)
  ✓ CCI
  ✓ Choppiness Index (CI > 61.8 = ranging, so DISABLE this — you want ranging)
  ✓ Vix Fix (for panic detection)
  ✓ Session Filter (0900-1600)
Switch Board: Squeeze Dots ✓, Market Sessions ✓
Signal Expiry: 2
Alternate Signal: Off
```

**Logic**: Vix Fix detects panic bottoms for buying opportunities. Stochastic detects overbought/oversold for reversal entries. RSI and CCI add momentum reversal confirmation. Session filter keeps you out of low-liquidity periods.

### Recipe 6: Multi-Timeframe Trend

```
Preset: Custom
Leading Indicator: Trend Meter
Trend Meter Settings:
  TM1: MACD Crossover - Fast - 8, 21, 5
  TM2: RSI 13: > or < 50
  TM3: RSI 5: > or < 50
  Alignment Mode: 3 TM + 2 TB Align
Confirmation Filters:
  ✓ EMA Filter (200)
  ✓ SMC HTF Trend (set to "D" for daily)
  ✓ Session Filter
Signal Expiry: 3
```

**Logic**: Trend Meter requires multiple trend assessment methods to agree before firing. EMA 200 adds long-term trend filter. SMC HTF on daily ensures you're aligned with the daily institutional trend. Session filter keeps trades during liquid hours.

---

## 14. Troubleshooting & FAQ

### Why am I not getting any signals?

1. **Too many confirmation filters enabled**: Each additional filter makes it harder for signals to pass. Try disabling all filters, then add them back one at a time.
2. **Signal expiry too low**: If expiry is 1-2 bars, confirmations may not have time to align. Try increasing to 5.
3. **Leading indicator doesn't match market conditions**: Range Filter won't fire much in a strong trend. Supertrend won't fire in a range. Match your leading indicator to the market.
4. **Alternate Signal mode**: If the last signal was long, you won't get another long until a short fires. Try turning it off temporarily to see if signals exist.

### Why am I getting too many signals?

1. **Too few confirmation filters**: Add more to increase selectivity.
2. **Signal expiry too high**: Lower to 1-2 bars for fresher signals only.
3. **Wrong leading indicator**: Some indicators fire frequently (RSI, MACD) while others are more selective (SMC, Breakout Channels). Switch to a more selective one.

### The dashboard shows X marks on filters I enabled

This means the filter is active but its current state doesn't support the trade direction. For example, if you enable "EMA Filter" and it shows X for Long, it means price is currently below the EMA — the filter is correctly blocking long signals until price moves above.

### SMC labels are cluttering my chart

1. Switch SMC Mode to "Present" — this shows only the latest structure instead of all historical labels
2. Reduce the SMC Swing Length to show fewer but more significant structures
3. Toggle off individual SMC elements (OBs, FVGs, Equal H/L) via the SMC settings

### Breakout Channels are disappearing

The script caps at 30 active channels to conserve TradingView's 500-box limit (shared with SMC). On very long charts, older channels are removed. This is by design — if you need more channels, reduce SMC overlays or shorten your chart view.

### Pine Script compilation error

If you get compilation errors after modifying the script, check:
- `plotshape` and `plot` must be at global scope (not inside `if` blocks)
- Functions must be defined at global scope
- `ta.*` functions must execute every bar (can't gate behind `if`)
- Variables declared with `var` persist across bars — make sure you reset them when needed

### Can I use this for automated trading?

TradingView requires a `strategy()` script (not `indicator()`) for automated trading via Strategy Tester. The AIO is an `indicator()`. However, the data window outputs (visible in TradingView's Data Window panel) expose signal values that a separate `strategy()` script can read via `request.security()`.

---

## 15. Pine Script Limitations

Understanding these constraints helps you set realistic expectations:

| Limitation | Impact | Workaround |
|-----------|--------|-----------|
| **500 labels/lines/boxes max** | SMC + BC overlays share this budget | SMC Present mode, BC capped at 30 channels |
| **`plotshape` must be at global scope** | Can't conditionally show/hide plotshapes | Use the condition itself (e.g., `switch_squeeze and sqzOn`) |
| **`ta.*` must execute every bar** | SMC calculations run even when disabled | Minimal overhead; calculations are lightweight |
| **Can't change `input.*` values at runtime** | Presets can't modify input toggles directly | Uses `eff_respect*` effective variables that combine input + preset |
| **`request.security` adds delay** | HTF SMC confluence has a 1-bar delay on the current timeframe | Acceptable for confirmation filters (state-based, not time-critical) |
| **No sub-panes in overlay scripts** | STC can't show as a separate oscillator pane | Uses `bgcolor()` for zone visualization + circle markers |
| **500 max array elements** | Limits historical SMC structure tracking | `var` + delete pattern keeps only what's needed |

---

## Appendix: Quick Reference Card

### Input Groups (Settings Panel Order)

1. **Indicator Setup** — Expiry, alternate, scoring, dashboard, presets
2. **Main Indicator (signal)** — Leading indicator dropdown
3. **Confirmation Indicators (filter)** — All 38 respect* toggles + sub-options
4. **Switch Board** — All 22 visual overlay toggles
5. **Fibonacci Display** — Fib settings
6. **MAs Line** — Moving average line settings
7. **Pivot Levels** — Pivot point settings
8. **Fractal** — Fractal detection settings
9. **Range Filter** — Range Filter parameters
10. **RQK** — Rational Quadratic Kernel parameters
11. **TSI** — True Strength Indicator parameters
12. **SuperTrend** — SuperTrend parameters
13. **HalfTrend** — HalfTrend parameters
14. **Trendline Breakout** — Trendline breakout parameters
15. **Ichimoku** — Ichimoku parameters
16. **SuperIchi** — SuperIchi parameters
17. **Donchian Channel Ribbon** — Donchian parameters
18. **DMI** — Directional Movement Index parameters
19. **Parabolic SAR** — PSAR parameters
20. **TDFI** — Trend Direction Force Index parameters
21. **McGinley Dynamic** — McGinley parameters
22. **CCI** — Commodity Channel Index parameters
23. **B-Xtrender** — B-Xtrender parameters
24. **VWAP** — VWAP parameters
25. **Chandelier Exit** — Chandelier parameters
26. **ROC** — Rate of Change parameters
27. **SSL Channel** — SSL parameters
28. **Chaiken Money Flow** — CMF parameters
29. **Vortex Index** — Vortex parameters
30. **Waddah Attar Explosion** — WAE parameters
31. **Range Detector** — Range Detector parameters
32. **Volatility Oscillator** — VO parameters
33. **DPO** — Detrended Price Oscillator parameters
34. **HA Candlestick Oscillator** — HACOLT parameters
35. **Choppiness Index** — CI parameters
36. **Damiani Volatmeter** — DV parameters
37. **MACD** — MACD parameters
38. **Awesome Oscillator** — AO parameters
39. **Wolf Pack ID** — Wolfpack parameters
40. **Bollinger Band** — BB overlay parameters
41. **BB Oscillator** — BB oscillator parameters
42. **Trend Meter** — TM meters, bars, alignment mode, MA settings
43. **Stochastic** — Stochastic parameters
44. **RSI** — RSI parameters
45. **HullSuite** — Hull Suite parameters
46. **Schaff Trend Cycle** — STC parameters + background transparency
47. **PVSRA** — PVSRA parameters
48. **Supply/Demand Zone** — POI parameters
49. **Heiken-ashi candles** — HA overlay parameters
50. **Fair Value Gap** — FVG parameters
51. **Liquidity Zone** — Liquidity zone parameters
52. **Market Sessions** (7 groups) — London, NY, Tokyo, HK, Sydney, EU Brinks, US Brinks
53. **QQE** — QQE Mod parameters
54. **Up/Down Volume** — Volume parameters
55. **Smart Money Concepts** — SMC parameters
56. **Breakout Channels** — BC parameters
57. **Williams Vix Fix** — Vix Fix parameters
58. **Squeeze Momentum** — Squeeze parameters

### Total Input Count: ~400+ inputs across 58 groups
