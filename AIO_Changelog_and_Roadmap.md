# AIO Strategy Builder — Changelog & Improvement Roadmap

## Script Info
- **File**: `Merged_All_in_One.pine`
- **Base**: DIY Custom Strategy Builder [ZP] v1 (4468 lines, Pine Script v6)
- **Current**: 5141 lines
- **Version**: Pine Script v6

---

## Changelog

### v3 — Feature Improvements (Current)

#### New Features

| # | Feature | Details |
|---|---------|---------|
| 1 | Multi-Timeframe SMC Confluence | `respectsmc_htf` confirmation filter using `request.security` for HTF swing trend bias. Configurable timeframe (default 4H). |
| 2 | SMC Order Block Mitigation Alerts | `respectob_mit` confirmation filter + alertconditions when bearish/bullish OBs are mitigated. |
| 3 | Squeeze Momentum "Squeeze Not Active" Filter | `respectsqz_active` confirmation filter — only allows signals when market is NOT in a squeeze. |
| 4 | Dashboard Enhancement | `show_detailed_dashboard` input adds a second table showing SMC/BC/Vix/Squeeze/EMA/STC status. |
| 5 | SMC Present Mode Label Optimization | Existing `var`+`delete` pattern already optimized — no change needed. |
| 6 | Signal Strength Scoring | `use_signal_scoring` mode replaces AND-chain with weighted scoring. Configurable `signal_score_threshold` (1-10). Backward compatible (AND-chain is default). |
| 7 | Backtest-Friendly Signal Outputs | 11 `plot(..., display=display.data_window)` outputs for external strategy consumption: long/short signals, scores, SMC/HTF/BC/Vix/Squeeze/STC bias, session state. |
| 8 | Trend Meter Implementation | 3 configurable trend meters + 2 trend bars + 3 alignment modes. Leading indicator + `respecttm` confirmation filter. Replaces ~330 lines of commented-out code with clean implementation. |
| 9 | Auto-Enable Related Overlays | `eff_switch_stc`/`eff_switch_squeeze` effective variables auto-enable STC/Squeeze display when selected as leading indicator. SMC/BC already auto-enabled via `or leadingindicator ==` guards. |
| 10 | Presets System | `preset` dropdown (Custom/SMC Scalper/Trend Follower/Breakout Trader/All Filters). Uses `eff_respect*` effective variables to override filter settings without modifying `input.*` values. |
| 11 | FVG-Order Block Confluence | `respectfv_ob` confirmation filter detects spatial overlap between Fair Value Gaps and Order Blocks. Alertconditions for bullish/bearish confluence. |
| 12 | Session-Based Filter | `respectsession` confirmation filter + `session_time` input (default 0900-1600). Uses `time(timeframe.period, session_time)`. |
| 13 | EMA Cross Display | `show_ema_cross` toggle + crossover/under plotshape markers + alertconditions when fast MA crosses slow MA. |
| 14 | STC Display Enhancement | `stc_bg_transparency` input + `bgcolor()` for green/red/orange STC zones on chart. Enhanced circle markers using `eff_switch_stc`. |

#### Architecture Changes

| # | Change | Details |
|---|--------|---------|
| 1 | Effective variables for presets | All `respect*` inputs have `eff_respect*` counterparts that combine input + preset override. Confirmation chains, scoring, and pushConfirmation all use `eff_respect*`. |
| 2 | Effective variables for display | `eff_switch_stc`/`eff_switch_squeeze` combine switch input + leading indicator selection for display elements (plotshape, bgcolor). |
| 3 | Dual signal variables | New indicators use `_conf_` suffixed variables for state-based confirmation vs event-based leading (consistent with v2 pattern). |

#### New Inputs

| Input | Group | Default | Purpose |
|-------|-------|---------|---------|
| `preset` | Setup | "Custom" | Pre-configured filter combinations |
| `use_signal_scoring` | Setup | false | Enable weighted scoring mode |
| `signal_score_threshold` | Setup | 3 | Minimum filters needed in scoring mode |
| `show_detailed_dashboard` | Setup | false | Show indicator status table |
| `show_ema_cross` | Switch Board | true | EMA crossover markers |
| `stc_bg_transparency` | STC | 90 | Background coloring transparency |
| `respectsmc_htf` | Confirmation | false | HTF SMC trend filter |
| `smc_htf_timeframe` | Confirmation | "240" | HTF timeframe for SMC |
| `respectob_mit` | Confirmation | false | OB mitigation filter |
| `respectfv_ob` | Confirmation | false | FVG-OB confluence filter |
| `respectsqz_active` | Confirmation | false | Squeeze not active filter |
| `respectsession` | Confirmation | false | Session time filter |
| `session_time` | Confirmation | "0900-1600" | Session window |
| `respecttm` | Confirmation | false | Trend Meter filter |
| `tm_mode` | Trend Meter | "3 TM Align" | Alignment mode |
| `TrendBar1-5` | Trend Meter | Various | Meter/bar selections |
| `MA1-4_Length/Type` | Trend Meter | Various | Trend bar MA settings |

#### New Alert Conditions

| Alert | Trigger |
|-------|---------|
| SMC HTF Trend Bullish/Bearish | HTF swing trend changes direction |
| SMC Bearish/Bullish OB Mitigated | Order block price mitigation |
| FVG-OB Bullish/Bearish Confluence | FVG overlaps with OB |
| EMA Cross Up/Down | Fast MA crosses slow MA |
| Trend Meter Bullish/Bearish Align | Meters/bars align in one direction |

#### Bug Fixes (Critical)

| # | Issue | Location | Fix |
|---|-------|----------|-----|
| 1 | QQE Mod leading indicator short signal used `isqqeabove` instead of `isqqebelow` — short signals were identical to long signals | Line ~4607 | Changed to `leadingshortcond := isqqebelow` |
| 2 | Breakout Channels forward iteration during `array.remove(i)` caused index corruption — items after removal were skipped, potential out-of-bounds | Lines 4404-4421 | Changed to backwards iteration: `for i = bc_boxes.size() - 1 to 0` |
| 3 | Vix Fix `vix_short = not vix_elevated` was true on every non-elevated bar — confirmation filter always passed shorts | Lines 4451-4468 | Split into transition-based leading signals (`vix_long`/`vix_short` fire on crossovers) and state-based confirmation signals (`vix_conf_long`/`vix_conf_short`) |
| 4 | Squeeze Momentum `sqz_long`/`sqz_short` only true on squeeze-release bars — confirmation filter only passed on a single bar | Lines 4484-4517 | Added `sqz_conf_long`/`sqz_conf_short` using momentum direction (`sqz_val > 0` / `< 0`) for confirmation; kept squeeze-release triggers for leading |
| 5 | SMC signals were state-based (`smc_long = bias == BULLISH` true every bar) — leading indicator fired continuously | Lines 4284-4355 | Split into event-based leading (`smc_trend_changed` + bias) and state-based confirmation (`smc_conf_long`/`smc_conf_short`) |
| 6 | `smc_currentAlerts` declared with `var` but never reset — once a flag was set to `true`, alerts would fire every bar forever | After line 4092 | Added `smc_currentAlerts := smc_alerts.new()` reset each bar |
| 7 | `bc_upbreak`/`bc_downbreak` declared inside `if` block but referenced by `plotshape` outside — scope error | Lines 4372-4373 | Moved declarations to global scope before the `if` block |
| 8 | `sqzOn`/`sqzOff` declared inside `if` block but referenced by `plotshape` outside — scope error | Lines 4489-4490 | Moved declarations to global scope before the `if` block |
| 9 | `bc_canCreate` function defined inside `if` block — Pine Script requires functions at global scope | Line 4375 | Moved function definition to global scope before the `if` block |
| 10 | `plotshape` calls inside `if` blocks — Pine Script v6 requires `plotshape`/`plot` at global scope | Lines 4428-4429, 4498-4501 | Moved all `plotshape` calls outside their `if` blocks |

#### Signal Quality Improvements

| # | Change | Details |
|---|--------|---------|
| 1 | SMC leading indicator now event-based | Only fires on trend change transitions, not on every bar where trend is bullish/bearish. Signal expiry system handles the rest. |
| 2 | SMC confirmation filter remains state-based | `smc_conf_long` is true on every bar where swing trend is bullish — correctly allows all longs during bullish trend |
| 3 | Vix Fix leading fires on transitions | `vix_long` fires when Vix Fix first crosses above threshold (bottom detection). `vix_short` fires when Vix Fix drops back below (exhaustion). |
| 4 | Vix Fix confirmation uses persistent bias | `vix_conf_long = not vix_elevated` (no panic = safe to long). `vix_conf_short = vix_elevated` (panic = caution, but potential bottom). |
| 5 | Squeeze confirmation uses momentum direction | `sqz_conf_long = sqz_val > 0` (positive momentum = bullish bias). `sqz_conf_short = sqz_val < 0` (negative momentum = bearish bias). Works every bar, not just on release. |

#### Performance Optimizations

| # | Change | Details |
|---|--------|---------|
| 1 | Breakout Channels capped at 30 active channels | `while bc_boxes.size() > 30` removes oldest channels. Prevents box budget exhaustion when SMC is also active (500 box limit shared). |
| 2 | `ta.atr(1)` replaced with `ta.tr` | Choppiness Index (line 2218) and STR calculation (line 1934) used `ta.atr(1)` which is functionally identical to `ta.tr` but slightly more expensive. |

#### New Features

| # | Feature | Details |
|---|---------|---------|
| 1 | BC "New Channel Formed" alert | `bc_newChannel` tracking variable + `alertcondition` added. Was in original AlgoAlpha script but lost during merge. |
| 2 | Vix Fix "Exhaustion Signal" alert | `alertcondition(vix_short, 'Vix Fix Exhaustion Signal', 'Vix Fix dropped from elevated - exhaustion')` |
| 3 | Input tooltips for all new indicators | Added descriptive tooltips to all SMC (9 inputs), Breakout Channels (7 inputs), Vix Fix (6 inputs), and Squeeze Momentum (5 inputs) settings. |

#### Code Quality

| # | Change | Details |
|---|--------|---------|
| 1 | Removed dead Trend Meter code | Commented-out `tmup`/`tmdown` variables and references removed from lines ~4741-4752, 4762, 4768, 4897. |
| 2 | Confirmation filter chains use `_conf_` variables | `longCond`/`shortCond` chains and `pushConfirmation` calls now use `smc_conf_long`, `vix_conf_long`, `sqz_conf_long` etc. for consistency. |
| 3 | Indicator declaration includes resource limits | `max_labels_count=500, max_lines_count=500, max_boxes_count=500` added to handle SMC + BC overlays. |

---

### v1 — Initial Merge

Merged 8 Pine Scripts into DIY Custom Strategy Builder base:

| Source Script | Integration |
|---------------|-------------|
| Smart Money Concepts [LuxAlgo] | Full overlay + leading indicator + confirmation filter. BOS/CHoCH, Order Blocks, FVG, Strong/Weak H/L, Equal H/L. |
| Smart Money Breakout Channels [AlgoAlpha] | Overlay + leading indicator + confirmation filter. Consolidation channels with breakout signals. Volume gauge removed to save resources. |
| CM_Williams_Vix_Fix | Leading indicator + confirmation filter. No overlay (dashboard only). |
| Squeeze Momentum Indicator [LazyBear] | Leading indicator + confirmation filter. Optional chart dots via switchboard. |
| EMA 9-21-50 + VWAP + MACD + RSI Pro [v6] | Not integrated — DIY builder already has all components individually. |
| Parabolic SAR + EMA 200 + MACD Signals | Not integrated — DIY builder already has PSAR, EMA filter, MACD as separate filters. |
| CM_Stochastic_MTF | Not integrated — DIY builder already has Stochastic. |

**4 new leading indicators** added to dropdown: Smart Money Concepts, Breakout Channels, Williams Vix Fix, Squeeze Momentum.

**4 new confirmation filters**: `respectsmc`, `respectbc`, `respectvix`, `respectsqz`.

**3 new switchboard toggles**: `switch_smc`, `switch_bc`, `switch_squeeze`.

**11 new alert conditions**: SMC BOS/CHoCH (4), SMC Equal H/L (2), BC Breakout (2), Vix Fix (1), Squeeze Release (2).

---

## Known Issues & Limitations

### Pine Script Platform Constraints
- `ta.*` functions (like `ta.atr(200)` for SMC) must execute every bar — they cannot be gated behind `if` blocks. This means SMC volatility calculations run even when SMC is disabled.
- `plotshape`/`plot` must be at global scope — cannot be inside conditional blocks.
- TradingView limits: 500 labels, 500 lines, 500 boxes per script. When SMC overlay + Breakout Channels are both active, they share this budget.

### Current Signal Behavior
- **SMC leading**: Fires only on trend change (event-based). After the first bar of a new trend, the leading indicator returns to false until the next trend change. The DIY builder's signal expiry system (default 3 bars) handles this correctly.
- **Breakout Channels leading**: Bullish/bearish breakout is a single-bar event. The signal disappears after the breakout bar. Signal expiry system handles this.
- **BC channel cap**: Oldest channels are removed when count exceeds 30. On very long charts, older channels will be lost.
- **Damiani Volatmeter** (`respectdv`) uses `dvup` for both long and short conditions — this is intentional (it's a "is trending" filter, not directional).
- **Choppiness Index** (`respectci`) uses `ci_filter` for both long and short — same reason (is trending filter).

### Dashboard
- Dashboard only shows confirmation filter status. No separate rows for SMC trend bias, BC channel status, Vix Fix state, or Squeeze state.

---

## Future Improvement Roadmap

### Priority 1 — High Impact

#### 1. Multi-Timeframe SMC Confluence Filter
Many pro traders use SMC on higher timeframes for bias and lower timeframes for entries.
- Add `smc_htf_timeframe` input (default "240" = 4H)
- Use `request.security` to get swing trend from higher TF
- Add `respectsmc_htf` confirmation filter: confirms long when HTF trend is bullish
- Adds 1 `request.security` call
- **Effort**: ~30 lines

#### 2. SMC Order Block Mitigation Alerts
The script detects when order blocks are mitigated in `smc_deleteOBs()` but doesn't fire alerts.
- Add `alertcondition` for OB mitigation
- Bullish OB broken = bearish signal, bearish OB broken = bullish signal
- Optional `respectob_mit` confirmation filter
- **Effort**: ~15 lines

#### 3. Squeeze Momentum Direction Filter
Add a separate confirmation filter `respectsqz_dir` that checks momentum direction regardless of squeeze state. More useful than current `respectsqz` which is momentum-direction based already (renamed to `sqz_conf_*`).
- Current `respectsqz` already uses momentum direction for confirmation
- Could add a "squeeze active" filter instead — only allow signals during squeeze (anticipation of breakout)
- **Effort**: ~10 lines

### Priority 2 — Medium Impact

#### 4. Dashboard Enhancement
Add optional detailed status rows for new indicators:
- SMC: current swing trend bias (Bullish/Bearish)
- Breakout Channels: In Channel / Breakout
- Vix Fix: Elevated / Normal
- Squeeze: Squeezing / Released Bull / Released Bear
- Controlled by a `show_detailed_dashboard` toggle
- **Effort**: ~30 lines

#### 5. SMC Present Mode Label Optimization
In "Present" mode, SMC only shows the latest structure, but still creates and deletes labels for historical bars before discarding them. Add early returns in `smc_drawLabel` and `smc_drawStructure` to skip label creation for old bars when in Present mode.
- **Effort**: ~10 lines

#### 6. Signal Strength Scoring
Instead of binary pass/fail for confirmation filters, add a weighted scoring system:
- Each enabled confirmation filter that passes adds to a score
- Configurable threshold (e.g., "at least 3 of 5 enabled filters must pass")
- Display score in dashboard
- **Effort**: ~50 lines

#### 7. Backtest-Friendly Signal Mode
Add a `strategy.entry`/`strategy.close` mode that allows using TradingView's Strategy Tester:
- Toggle between `indicator` and `strategy` mode via input
- Map `longCondition`/`shortCondition` to `strategy.entry` calls
- Add position sizing, stop loss, take profit inputs
- **Effort**: ~40 lines

### Priority 3 — Nice to Have

#### 8. Trend Meter Implementation
The Trend Meter leading indicator exists as commented-out dead code. Implement it properly:
- 3 configurable trend meters (MACD cross, RSI levels, trend candles)
- 2 configurable trend bars
- Multiple alignment modes
- **Effort**: ~100 lines

#### 9. Auto-Enable Related Overlays
When SMC is selected as leading indicator, auto-enable `switch_smc` overlay. When Breakout Channels is selected, auto-enable `switch_bc`. Currently users must enable both separately.
- **Effort**: ~10 lines

#### 10. Input Group Collapse
The script has 404 inputs across many groups. TradingView doesn't support collapsible input groups, but we could:
- Add a "Presets" input that sets common configurations (e.g., "SMC Scalper", "Trend Follower", "All Filters")
- Reduce visible inputs by moving rarely-used settings into "Advanced" sub-groups
- **Effort**: ~30 lines

#### 11. FVG Order Block Confluence
When a Fair Value Gap overlaps with an Order Block, it creates a high-probability zone. Add visual highlighting and a `respectfv_ob` confirmation filter.
- **Effort**: ~25 lines

#### 12. Session-Based Filter
Add a time-of-day filter (e.g., only trade during London/NY overlap). Common in institutional trading.
- Add `session_start` and `session_end` inputs
- `respectsession` confirmation filter
- **Effort**: ~15 lines

---

## Architecture Reference

### Signal Flow
```
Leading Indicator (1 of 39 options)
    ↓
Confirmation Filters (0-37 enabled toggles, all must pass)
    ↓
Signal Expiry (default 3 bars window)
    ↓
Alternate Signal Mode (prevents consecutive same-direction signals)
    ↓
longCondition / shortCondition → plotshape, alerts
```

### New Indicator Variable Naming Convention
| Indicator | Leading (event-based) | Confirmation (state-based) | Overlay Toggle |
|-----------|----------------------|---------------------------|----------------|
| SMC | `smc_long`, `smc_short` | `smc_conf_long`, `smc_conf_short` | `switch_smc` |
| Breakout Channels | `bc_bullishBreakout`, `bc_bearishBreakout` | (same — already single-bar events) | `switch_bc` |
| Vix Fix | `vix_long`, `vix_short` | `vix_conf_long`, `vix_conf_short` | None |
| Squeeze Momentum | `sqz_long`, `sqz_short` | `sqz_conf_long`, `sqz_conf_short` | `switch_squeeze` |

### Resource Budget
| Resource | Limit | Primary Consumer | Secondary |
|----------|-------|-----------------|-----------|
| Labels | 500 | SMC structure labels | — |
| Lines | 500 | SMC trailing H/L, center lines | BC center lines |
| Boxes | 500 | SMC order blocks + FVGs | BC channel boxes (capped at 30) |

### Key Line References
| Section | Lines | Description |
|---------|-------|-------------|
| Indicator declaration | 1-6 | Title, overlay, resource limits |
| Inputs | 7-400 | 404 total inputs across groups |
| SMC UDTs & functions | 4027-4282 | Types, constants, helper functions |
| SMC execution | 4283-4355 | Structure detection, drawing, signals |
| Breakout Channels | 4357-4430 | Channel detection, box management, signals |
| Vix Fix | 4440-4468 | WVF calculation, transition signals |
| Squeeze Momentum | 4483-4517 | BB/KC squeeze, momentum, chart dots |
| Leading indicator branches | 4530-4740 | if/else chain mapping dropdown to signals |
| longCond/shortCond chains | 4758-4768 | AND chain of all confirmation filters |
| Signal expiry | 4770-4810 | Bar counting, alternate signal mode |
| Dashboard | 4980-5028 | Table rendering |
| Alert conditions | 5033-5047 | SMC (6), BC (3), Vix Fix (2), Squeeze (2) |
