# AIO Sniper [SMC HUD] — Complete Documentation

## Overview

**AIO Sniper** is a minimalist, purpose-built Pine Script v6 indicator for TradingView, designed for **HSI (Hang Seng Index)** day traders who follow **Smart Money Concepts (SMC)**. It acts as a fighter-jet HUD: the chart stays transparent and clean by default, and only lights up when high-probability setups enter the kill zone.

**Philosophy:** Less is More. The script is the radar — you are the sniper who decides whether to pull the trigger.

---

## Quick Start

1. Go to [Raw file on GitHub](https://raw.githubusercontent.com/puzzithinker/pinescripts/main/AIO_Sniper.pine)
2. Copy the entire content
3. Open TradingView → Pine Editor → paste
4. Click "Add to chart"
5. Apply to an **HSI 1-minute or 5-minute chart**

> **Important:** Always copy from the Raw URL. Copying from GitHub's web interface corrupts indentation and causes "2 indicator declarations" errors.

---

## The 6 Core Features

---

### Feature 1: Auto Premium/Discount Zones

#### What It Does

Automatically identifies the most recent Swing High and Swing Low, then divides the range into two zones:

| Zone | Range | Background Color | Meaning |
|------|-------|-----------------|---------|
| **Premium** | 50% – 100% | Faint red | Price is expensive — Smart Money is distributing (selling) |
| **Discount** | 0% – 50% | Faint green | Price is cheap — Smart Money is accumulating (buying) |

Three key levels are drawn as horizontal lines:
- **0%** (Swing Low) — "Discount" label
- **50%** (Midpoint) — "50%" label
- **100%** (Swing High) — "Premium" label

#### How It Helps You

This is the **first filter** before any trade. The rule is simple:

> **If you want to go LONG, wait for price to enter the Discount zone (green).**
> **If you want to go SHORT, wait for price to enter the Premium zone (red).**

When price is in the premium zone and you're tempted to buy, the red background is a visual warning: *"Stop! You're buying at retail price — Smart Money is selling to you."*

#### Settings (Premium/Discount Zones group)

| Setting | Default | Description |
|---------|---------|-------------|
| Show Premium/Discount Zones | true | Toggle zone display |
| Premium Zone Color | Red (92% transparent) | Background color for premium area |
| Discount Zone Color | Green (92% transparent) | Background color for discount area |
| 50% Midline Color | Gray (60% transparent) | Color of the midpoint line |
| 0%/100% Line Color | Gray (70% transparent) | Color of boundary lines |
| Filter OB/FVG by Zone | true | **Core feature** — only show zone-filtered OBs and FVGs |

---

### Feature 2: Smart OB & FVG Highlighter with Zone Filter

#### What It Does

Detects **Order Blocks (OB)** and **Fair Value Gaps (FVG)** using the SMC engine, but with a critical twist:

| Pattern | Display Condition | Why |
|---------|-------------------|-----|
| **Bullish OB** | Only if it overlaps the **Discount zone** (below 50%) | Buying interest at wholesale price = high probability |
| **Bearish OB** | Only if it overlaps the **Premium zone** (above 50%) | Selling interest at retail price = high probability |
| **Bullish FVG** | Only if it's in the **Discount zone** | Gap up from accumulation = Smart Money footprint |
| **Bearish FVG** | Only if it's in the **Premium zone** | Gap down from distribution = Smart Money footprint |

All other OBs and FVGs are **hidden**. This is the key differentiator — your chart shows only high-probability Points of Interest (POIs), not every single gap and block.

#### How It Helps You

Without the filter, SMC scripts paint 20-30 OBs and FVGs on the chart, making it unreadable. With the filter:

- You only see **the OBs where Smart Money actually left footprints at good prices**
- Your chart stays clean — only 2-3 POIs visible at any time
- These filtered zones become your **entry zones** — wait for price to return here

#### Visual Elements

- **Bullish OB**: Blue box (swing-level) or lighter blue box (internal-level), extending right
- **Bearish OB**: Dark red box (swing-level) or lighter red box (internal-level), extending right
- **Bullish FVG**: Green semi-transparent box
- **Bearish FVG**: Red semi-transparent box

#### Settings (Smart Money Concepts group)

| Setting | Default | Description |
|---------|---------|-------------|
| Show Order Blocks | true | Toggle OB display |
| Show Fair Value Gaps | true | Toggle FVG display |
| Max OBs Displayed | 5 | Number of OB boxes to render (keeps chart clean) |
| Swing Length | 50 | Bars to look back for swing point detection |
| Internal Structure Length | 5 | Bars for internal (minor) structure detection |
| Style | Colored | "Colored" for green/red, "Monochrome" for grayscale |
| Display Mode | Present | "Present" = only current structure; "Historical" = all past structures |

---

### Feature 3: Market Structure — BOS & CHoCH Labels

#### What It Does

Automatically labels **Break of Structure (BOS)** and **Change of Character (CHoCH)** on the chart:

| Label | Meaning | Trading Implication |
|-------|---------|-------------------|
| **BOS** | Price broke the previous swing in the **same direction** as the trend | Trend continuation — look for pullback entries |
| **CHoCH** | Price broke the previous swing in the **opposite direction** from the trend | Potential trend reversal — this is your trigger signal |

The script detects both:
- **Swing structure** (major moves, use on 5m chart for macro direction)
- **Internal structure** (minor moves, use on 1m chart for entry timing)

#### How It Helps You

**On the 5-minute chart:**
- BOS confirms the macro direction. If you see bullish BOS, your bias is LONG.
- CHoCH warns of a potential reversal. Start looking for counter-trend setups.

**On the 1-minute chart (the trigger):**
- When price enters your filtered OB zone in the discount area, wait for a **bullish CHoCH** label to appear — that's your signal to go LONG.
- When price enters your filtered OB zone in the premium area, wait for a **bearish CHoCH** label — that's your signal to go SHORT.

#### Visual Elements

- **Swing BOS/CHoCH**: Solid line connecting the broken level to the current bar, with a label
- **Internal BOS/CHoCH**: Dashed line with a smaller label
- Colors: Green for bullish structure, Red for bearish structure (or grayscale in Monochrome mode)

---

### Feature 4: Liquidity Pools / Targets

#### What It Does

Draws horizontal lines at key liquidity levels where Smart Money's stop losses are clustered — these are **magnets** that price tends to gravitate toward:

| Level | Label | Description |
|-------|-------|-------------|
| **Previous Day High** | PDH | Yesterday's highest price — buy stops are trapped above |
| **Previous Day Low** | PDL | Yesterday's lowest price — sell stops are trapped below |
| **Session High** | Session H | Current trading session's highest price |
| **Session Low** | Session L | Current trading session's lowest price |
| **Equal Highs** | EQH | Two swing highs at approximately the same level — buy stop liquidity pool |
| **Equal Lows** | EQL | Two swing lows at approximately the same level — sell stop liquidity pool |

#### How It Helps You

These levels serve two purposes:

1. **Entry Confirmation:** Smart Money often sweeps these liquidity levels before reversing. If price sweeps PDH then reverses with a CHoCH into a discount OB — that's a textbook long setup.

2. **Take Profit Targets:** After entering a trade, these levels are your **most precise TP targets**. Price will naturally gravitate toward the nearest liquidity pool. Set your TP1 at the nearest EQH/EQL or PDH/PDL.

#### Settings (Liquidity Targets group)

| Setting | Default | Description |
|---------|---------|-------------|
| Show PDH/PDL | true | Toggle Previous Day High/Low lines |
| PDH/PDL Color | Orange (40% transparent) | Color of PDH/PDL lines |
| Show Session H/L | true | Toggle current session High/Low lines |
| Session H/L Color | Blue (40% transparent) | Color of session H/L lines |

---

### Feature 5: Kill Zones & Garbage Time

#### What It Does

Colors the chart background based on HSI's trading session in Hong Kong Time (HKT / GMT+8):

| Time (HKT) | Zone | Background Color | What You Should Do |
|-------------|------|-------------------|-------------------|
| 09:30 – 11:30 | **Morning Kill Zone** | Faint green | Radar ON — highest volume & momentum of the day |
| 11:30 – 13:30 | **Lunch Garbage** | Gray | Radar OFF — low volume, choppy, fake breakouts |
| 13:30 – 15:00 | **Afternoon Kill Zone** | Faint yellow | Radar ON — European open, new capital flows in |
| 15:00 – 16:00 | **Close Garbage** | Faint red | Radar OFF — forced closing, random spikes, DO NOT open new positions |

#### How It Helps You

This is a **behavioral guardrail**. Even if the chart shows a perfect setup during garbage time, the gray/red background reminds you: *"This is a trap. Smart Money isn't active right now. Don't trade."*

The morning kill zone (09:30-11:30) is where Smart Money typically:
- Sweeps the previous day's liquidity (PDH/PDL)
- Establishes the daily bias via BOS or CHoCH
- Creates the OBs and FVGs you'll trade from

#### Settings (Kill Zones group)

| Setting | Default | Description |
|---------|---------|-------------|
| Show Kill Zones | true | Master toggle |
| Morning KZ session | "0930-1130" | Time range for morning kill zone |
| Morning KZ color | Green (92% transparent) | Background color |
| Morning KZ enabled | true | Toggle this specific zone |
| Lunch Garbage session | "1130-1330" | Time range for lunch garbage |
| Lunch Garbage color | Gray (92% transparent) | Background color |
| Lunch Garbage enabled | true | Toggle |
| Afternoon KZ session | "1330-1500" | Time range for afternoon kill zone |
| Afternoon KZ color | Yellow (92% transparent) | Background color |
| Afternoon KZ enabled | true | Toggle |
| Close Garbage session | "1500-1600" | Time range for close garbage |
| Close Garbage color | Red (92% transparent) | Background color |
| Close Garbage enabled | true | Toggle |
| Timezone | Asia/Hong_Kong | Timezone for session calculation |

> **Tip:** You can customize the session strings to match different markets (e.g., change to `"0930-1600"` for full-day mode, or adjust for US market hours with `"America/New_York"` timezone).

---

### Feature 6: R:R & Risk Dashboard

#### What It Does

When a sniper signal fires, the dashboard automatically:

1. **Marks the entry price** with a white horizontal line
2. **Calculates the Stop Loss (SL)** using your chosen mode
3. **Identifies the Take Profit (TP1)** as the nearest liquidity target
4. **Draws SL and TP lines** on the chart (red dashed = SL, green dashed = TP)
5. **Displays a table** with all key metrics

#### Stop Loss Logic — Buffered Structural SL

The SL is never placed at a random ATR offset. It's placed at a **structural level** with a buffer to survive stop hunts:

**Mode A: Aggressive (OB + Buffer)**
- For LONG: SL = lowest point of the nearest bullish OB – buffer points
- For SHORT: SL = highest point of the nearest bearish OB + buffer points
- Best for: Maximum R:R ratio

**Mode B: Conservative (Swing + Buffer)**
- For LONG: SL = Swing Low – buffer points
- For SHORT: SL = Swing High + buffer points
- Best for: Higher win rate, wider stops

**Smart Toggle:**
If the OB edge and the Swing point are within 30 points (configurable) of each other, the script automatically switches to the Swing-based SL. Why? Because when they're that close, the Swing point is the stronger structural defense — using the OB would give almost the same SL but with less structural significance.

**Buffer:**
Default 20 points for HSI. This protects against "stop hunts" (wicks that briefly pierce the OB before reversing). Never place your SL exactly at the OB boundary — that's where liquidity is densest.

#### Take Profit Logic

TP1 is set to the nearest liquidity magnet:
- For LONG: PDH (previous day high) or Swing High
- For SHORT: PDL (previous day low) or Swing Low

#### Dashboard Table

```
┌─────────────┬──────────────┐
│ Direction   │ LONG         │
│ Entry       │ 19,245       │
│ SL          │ 19,200 (-45) │
│ TP1         │ 19,350 (+105)│
│ R:R         │ 1:2.3 ✅     │
│ Size        │ 2.22 lots    │
│ Mode        │ Aggressive   │
└─────────────┴──────────────┘
```

- **R:R color**: Green if ≥ minimum R:R (default 1:2), Red if below — this is your **go/no-go signal**
- **Size**: Calculated as `(Account × Risk%) / (Entry - SL)` — tells you exactly how many lots to trade

#### Settings (R:R Dashboard group)

| Setting | Default | Description |
|---------|---------|-------------|
| Show R:R Dashboard | true | Toggle the table |
| SL Mode | Aggressive (OB+Buffer) | Choose between Aggressive and Conservative |
| SL Buffer (points) | 20 | Points added below/above structural level for stop hunt protection |
| Smart Toggle Distance (pts) | 30 | If OB and Swing are within this distance, auto-switch to Swing-based SL |
| Account Size | 100,000 | Your trading account balance |
| Risk % | 1.0 | Percentage of account risked per trade |
| Minimum R:R | 2.0 | R:R below this threshold shows red warning |
| Dashboard Position | bottom_right | Corner of the chart for the table |

---

## Signal Logic

### Sniper Long Signal

Fires when **ALL** of these conditions are met simultaneously:

1. **Price is in the Discount zone** (close < 50% midpoint)
2. **Bullish CHoCH** is detected on swing structure (trend reversal confirmed)

### Sniper Short Signal

Fires when **ALL** of these conditions are met simultaneously:

1. **Price is in the Premium zone** (close > 50% midpoint)
2. **Bearish CHoCH** is detected on swing structure (trend reversal confirmed)

### Why This Works

This is the SMC "confluence" approach distilled to its purest form:

- **Zone filter** ensures you're entering at wholesale price (discount) or retail price (premium)
- **CHoCH** ensures Smart Money has actually reversed direction — you're not catching a falling knife
- **No oscillators, no MACD, no RSI** — pure price action and structure

---

## Alerts

The script provides 6 configurable alerts:

| Alert Name | Trigger | Use Case |
|------------|---------|----------|
| **Sniper Long** | Price in Discount + Bullish CHoCH | Your primary long entry signal |
| **Sniper Short** | Price in Premium + Bearish CHoCH | Your primary short entry signal |
| **Swing Bullish CHoCH** | Any swing bullish CHoCH | Early reversal warning |
| **Swing Bearish CHoCH** | Any swing bearish CHoCH | Early reversal warning |
| **Equal Highs** | EQH detected | Liquidity target above — potential short trap |
| **Equal Lows** | EQL detected | Liquidity target below — potential long trap |

### Setting Up Alerts

1. Right-click on chart → "Add Alert"
2. Select "AIO Sniper" as the condition source
3. Choose the alert condition from the dropdown
4. Set notification preferences (popup, email, push)

---

## Complete Trading Workflow

Here's how to use AIO Sniper in a real trading session:

### Step 1: Pre-Market (Before 09:30 HKT)
- Open HSI 5-minute chart
- Note PDH and PDL levels — these are today's liquidity magnets
- Identify the current swing structure bias

### Step 2: Morning Kill Zone (09:30 – 11:30 HKT)
- Watch for price to sweep PDH or PDL
- Wait for a **CHoCH** after the sweep
- Check: Is price in the **Discount zone** (green background)? → Prepare for LONG
- Check: Is price in the **Premium zone** (red background)? → Prepare for SHORT
- Wait for the **Sniper Long/Short** label to appear
- Check R:R dashboard: Is R:R green (≥ 1:2)? If yes, enter. If red, skip.

### Step 3: Lunch Garbage (11:30 – 13:30 HKT)
- **Do not trade.** The gray background is your guardrail.
- Manage existing positions only (trail stop, partial close).

### Step 4: Afternoon Kill Zone (13:30 – 15:00 HKT)
- Look for continuation setups or reversal setups
- Same rules: Discount + CHoCH = Long, Premium + CHoCH = Short

### Step 5: Close Garbage (15:00 – 16:00 HKT)
- **Do not open new positions.** The red background warns you.
- Close all remaining positions or trail stop to breakeven.

---

## Settings Reference

### Smart Money Concepts

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| Swing Length | int | 50 | Lookback for major swing points |
| Internal Structure Length | int | 5 | Lookback for minor swing points |
| Equal Highs/Lows Length | int | 3 | Lookback for equal H/L detection |
| Show Swing Structure | bool | true | Major BOS/CHoCH labels |
| Show Internal Structure | bool | true | Minor BOS/CHoCH labels |
| Show Order Blocks | bool | true | OB box display |
| Show Fair Value Gaps | bool | true | FVG box display |
| Show Equal H/L | bool | true | EQH/EQL dotted lines |
| Show Strong/Weak H/L | bool | true | Strong/Weak labels on trailing extremes |
| Max OBs Displayed | int | 5 | Keep chart clean |
| Style | string | Colored | Colored or Monochrome |
| Display Mode | string | Present | Present (current only) or Historical (all) |
| Swing Bullish | color | #089981 | Bullish structure color |
| Swing Bearish | color | #f23645 | Bearish structure color |
| Internal Bullish | color | #089981 | Internal bullish color |
| Internal Bearish | color | #f23645 | Internal bearish color |

### Premium/Discount Zones

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| Show Premium/Discount Zones | bool | true | Master toggle |
| Premium Zone Color | color | Red (92% trans) | Background above 50% |
| Discount Zone Color | color | Green (92% trans) | Background below 50% |
| 50% Midline Color | color | Gray (60% trans) | Dashed midpoint line |
| 0%/100% Line Color | color | Gray (70% trans) | Dotted boundary lines |
| Filter OB/FVG by Zone | bool | true | **Core filter** — only show zone-qualified OBs/FVGs |

### Kill Zones (HSI)

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| Show Kill Zones | bool | true | Master toggle |
| Morning KZ session | session | 0930-1130 | Morning kill zone time |
| Morning KZ color | color | Green (92% trans) | Background color |
| Morning KZ enabled | bool | true | Toggle |
| Lunch Garbage session | session | 1130-1330 | Lunch garbage time |
| Lunch Garbage color | color | Gray (92% trans) | Background color |
| Lunch Garbage enabled | bool | true | Toggle |
| Afternoon KZ session | session | 1330-1500 | Afternoon kill zone time |
| Afternoon KZ color | color | Yellow (92% trans) | Background color |
| Afternoon KZ enabled | bool | true | Toggle |
| Close Garbage session | session | 1500-1600 | Close garbage time |
| Close Garbage color | color |Red (92% trans) | Background color |
| Close Garbage enabled | bool | true | Toggle |
| Timezone | string | Asia/Hong_Kong | Session timezone |

### Liquidity Targets

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| Show PDH/PDL | bool | true | Previous Day High/Low lines |
| PDH/PDL Color | color | Orange (40% trans) | Line color |
| Show Session H/L | bool | true | Current session High/Low |
| Session H/L Color | color | Blue (40% trans) | Line color |

### R:R Dashboard

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| Show R:R Dashboard | bool | true | Master toggle |
| SL Mode | string | Aggressive (OB+Buffer) | Aggressive or Conservative |
| SL Buffer (points) | float | 20 | Buffer below/above structural SL |
| Smart Toggle Distance (pts) | float | 30 | Auto-switch threshold |
| Account Size | float | 100,000 | Your account balance |
| Risk % | float | 1.0 | Risk per trade as % of account |
| Minimum R:R | float | 2.0 | Green/red threshold for R:R |
| Dashboard Position | string | bottom_right | Chart corner |

### Signals

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| Show Entry Signals | bool | true | Toggle LONG/SHORT labels |

---

## Technical Notes

### Plot Budget

The script uses only **12 of 64** available plot slots:

| Function | Count | Slot Type |
|----------|-------|-----------|
| Kill Zone bgcolor | 1 | bgcolor (handles all 4 zones via ternary) |
| Sniper Long signal | 1 | plotshape |
| Sniper Short signal | 1 | plotshape |
| Alert: Sniper Long | 1 | alertcondition |
| Alert: Sniper Short | 1 | alertcondition |
| Alert: Bullish CHoCH | 1 | alertcondition |
| Alert: Bearish CHoCH | 1 | alertcondition |
| Alert: Equal Highs | 1 | alertcondition |
| Alert: Equal Lows | 1 | alertcondition |

All other visual elements (zone boxes, OB boxes, FVG boxes, structure lines, liquidity lines, entry/SL/TP lines, dashboard table) use `box.new()`, `line.new()`, `label.new()`, and `table` — which do **not** count toward the 64-plot limit.

### Performance

- Swing detection uses 50-bar lookback (adjustable) — sufficient for 5m charts
- OB and FVG arrays are capped at 50 entries, with oldest entries popped
- Only the most recent 5 OBs are rendered visually (adjustable up to 20)
- All heavy drawing operations only execute on `barstate.islast` or `barstate.islastconfirmedhistory`

### Compatibility

- **Pine Script v6** (`//@version=6`)
- Works on any timeframe but optimized for **1m and 5m HSI charts**
- The `Asia/Hong_Kong` timezone setting ensures kill zones align correctly regardless of your TradingView display timezone

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Scripts must contain one declaration statement" | Copy from the Raw GitHub URL, not the web interface |
| No signals appearing | Ensure both "Show Entry Signals" and "Show Premium/Discount Zones" are ON; signals require price to be in discount/premium zone |
| Kill zones not coloring | Check that "Show Kill Zones" is ON and your chart's timezone isn't conflicting — the script uses `Asia/Hong_Kong` internally |
| OBs not showing | Check "Show Order Blocks" is ON; with zone filter ON, bullish OBs only appear below the 50% midpoint |
| R:R dashboard shows "0" | The dashboard only populates after the first sniper signal fires — it's not a live position tracker |
| Volume error on some symbols | Some forex/index symbols don't provide volume data — the script will show a runtime error on those |

---

## File Location

- **Script:** `AIO_Sniper.pine` (in repository root)
- **Raw URL:** `https://raw.githubusercontent.com/puzzithinker/pinescripts/main/AIO_Sniper.pine`
- **Legacy script:** `Merged_All_in_One.pine` (still available, 5100+ lines, full indicator suite)