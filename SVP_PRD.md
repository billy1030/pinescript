# PRD — Smart Volume Profile (SVP)

## 1. Product Vision & Overview

**Indicator Name:** Smart Volume Profile [SVP]  
**Platform:** TradingView  
**Language:** Pine Script v6  
**Type:** Overlay Indicator with right-side histogram and dashboard panel  

SVP is a professional-grade volume profile indicator optimized for **high-beta stocks** (e.g., SOXL, TQQQ, NVDA). It goes far beyond the standard TradingView VRVP by combining institutional-grade volume analysis with smart money detection features including Volume Imbalance Zones, Low Volume Node (LVN) breakout alerts, multi-timeframe POC tracking, and Naked POC magnets. The indicator provides clear, actionable support/resistance levels derived from actual traded volume rather than arbitrary price levels.

---

## 2. Target Users

| Persona | Needs |
|---|---|
| **High-Beta Day Trader (5m–15m)** | LVN breakout zones for acceleration detection, Volume Imbalance for smart money footprints, fast visual S/R. |
| **Intraday Scalper** | POC as intraday gravity center, VAH/VAL for range-bound mean reversion targets. |
| **Swing Trader** | Multi-TF POC (Daily/Weekly) confluence zones, Naked POC as price magnets for target projection. |
| **Options Trader** | HVN for pinning levels (max pain equivalent), LVN for expected fast moves (gamma squeeze zones). |
| **AKRTS User** | Complementary to AKRTS — SVP provides WHERE to trade, AKRTS provides WHEN to trade. |

---

## 3. Core Concepts

### 3.1 Volume Profile Fundamentals

Volume Profile distributes total traded volume across price levels (rather than time), revealing the market's **accepted value** and **rejection zones**.

```
For each price level P in [Low, High]:
    Volume_at_P = Σ (volume of bars where Low ≤ P ≤ High)
```

| Concept | Definition | Trading Significance |
|---|---|---|
| **Point of Control (POC)** | Price level with the highest traded volume | Price magnet / fair value — price tends to gravitate here |
| **Value Area High (VAH)** | Upper boundary of 70% volume zone | Resistance / breakout level |
| **Value Area Low (VAL)** | Lower boundary of 70% volume zone | Support / breakdown level |
| **High Volume Node (HVN)** | Local peaks in the volume distribution | Price consolidation zones — strong S/R |
| **Low Volume Node (LVN)** | Local valleys in the volume distribution | Price acceleration zones — breakout highways |

---

## 4. Feature Catalog

### 4.1 Volume Profile Histogram

The core volume distribution display rendered as a horizontal histogram on the right side of the chart.

| Aspect | Detail |
|---|---|
| **Price Rows** | Configurable number of rows (24–200) dividing the visible price range evenly |
| **Volume Calculation** | For each row, sum all bar volumes where the bar's price range overlaps with the row's price range. Volume is proportionally allocated when a bar spans multiple rows. |
| **Buy/Sell Split** | Each row shows buy volume (green) and sell volume (red) side by side. Buy = bars where close > open. Sell = bars where close < open. |
| **Width Scaling** | Histogram bars scaled proportionally where the POC row = maximum configured width |
| **Lookback Modes** | Fixed bars, Session (Today), N Sessions, or Custom bar count |
| **Non-Repainting** | Uses confirmed bar data only (no future look-ahead) |

**Volume Allocation Formula (Proportional):**

```
For bar B with range [B.low, B.high] and row R with range [R.low, R.high]:
    overlap = max(0, min(B.high, R.high) - max(B.low, R.low))
    bar_range = B.high - B.low
    allocated_volume = (overlap / bar_range) × B.volume
```

---

### 4.2 POC, VAH, VAL Lines

Horizontal reference lines marking the key structural levels of the volume profile.

| Aspect | Detail |
|---|---|
| **POC Line** | Horizontal line at the price level with maximum volume. Extends across the full chart width. |
| **Value Area** | The price range containing approximately 70% of total traded volume (configurable 60%–80%). Calculated by expanding outward from the POC, alternating between the row above and below, adding whichever has more volume. |
| **VAH Line** | Top of the Value Area |
| **VAL Line** | Bottom of the Value Area |
| **Line Styles** | Each line has configurable color, width, and style (solid/dashed/dotted) |
| **Labels** | Optional price labels on each line showing the exact price level |

---

### 4.3 Volume Imbalance Detection (Smart Money Footprint)

Identifies price levels where aggressive buying or selling significantly dominates, revealing institutional positioning.

| Aspect | Detail |
|---|---|
| **Imbalance Ratio** | For each price row: `ratio = buy_volume / (buy_volume + sell_volume)` |
| **Buy Imbalance Zone** | Row where `ratio ≥ 0.70` (70%+ buy volume). Indicates aggressive accumulation. |
| **Sell Imbalance Zone** | Row where `ratio ≤ 0.30` (30%- buy volume / 70%+ sell volume). Indicates aggressive distribution. |
| **Threshold** | Configurable imbalance threshold (default 70%). Higher = more selective. |
| **Minimum Volume** | Imbalance zones require minimum volume above the Nth percentile to filter noise |
| **Visual** | Highlighted zone rectangles: bright green for buy imbalance, bright red for sell imbalance |
| **Alert** | Fire alert when current price enters a previously established imbalance zone |

**Trading Logic for High Beta:**
- Buy Imbalance Zone below current price = Strong institutional floor (support)
- Sell Imbalance Zone above current price = Strong institutional ceiling (resistance)
- Price entering Buy Imbalance from above = Expect bounce / accumulation hold
- Price entering Sell Imbalance from below = Expect rejection / distribution dump

---

### 4.4 LVN (Low Volume Node) Breakout Zones

Detects price levels with minimal traded volume — these are the "air pockets" where high-beta stocks accelerate through with minimal resistance.

| Aspect | Detail |
|---|---|
| **LVN Detection** | Price rows where volume is below the Nth percentile of all rows (default: below 20th percentile) |
| **Zone Width** | LVN zones can span multiple consecutive low-volume rows — merged into a single zone |
| **Breakout Alert** | Fire alert when price enters an LVN zone — expect fast directional movement |
| **Zone Direction Bias** | If the LVN zone is between two HVN zones, the bias is toward the HVN with higher volume (gravity pull) |
| **Visual** | Semi-transparent yellow/orange rectangles marking LVN zones |
| **Significance Score** | Each LVN is scored by the volume contrast between it and its neighboring HVN zones. Higher contrast = more explosive breakout potential. |

**Why This Matters for High Beta:**
```
SOXL price at $55 (HVN zone — lots of resting orders, slow movement)
LVN zone at $56-$57 (air pocket — almost no resting orders)  
HVN zone at $58 (lots of resting orders — next support/resistance)

If price breaks $55.50 → it will ACCELERATE through $56-$57 with minimal friction
→ Likely won't stop until it hits the $58 HVN
```

---

### 4.5 Multi-Timeframe POC

Simultaneously displays POC lines from multiple timeframes to identify confluence zones.

| Aspect | Detail |
|---|---|
| **Timeframes** | Current session, Daily, Weekly (all configurable) |
| **Calculation** | Each timeframe computes its own independent volume profile and POC |
| **Confluence Detection** | When 2+ POCs from different timeframes are within 0.5 ATR of each other, flag as "POC Confluence Zone" |
| **Visual** | Different colored/styled lines for each timeframe's POC. Confluence zones highlighted with a background fill. |
| **Data Source** | Uses `request.security()` for higher TF data with `lookahead = barmerge.lookahead_off` (non-repainting) |

---

### 4.6 Naked POC (nPOC) Tracker

Tracks previous session POCs that price has NOT yet revisited — these act as future price magnets.

| Aspect | Detail |
|---|---|
| **Definition** | A POC from a completed session whose price level has not been touched by any subsequent bar's high-low range |
| **Tracking** | Maintain a rolling list of up to N previous session POCs (default: 5) |
| **Pruning** | When price touches a Naked POC, it is considered "filled" and removed from the list |
| **Visual** | Dotted horizontal lines extending forward from the session where the POC originated |
| **Alert** | Fire alert when price approaches within 0.5 ATR of a Naked POC |
| **Magnet Effect Label** | Show the distance (in ATR units) to the nearest nPOC in the dashboard |

**Why Naked POCs Matter:**
Unfilled POCs represent zones of "unfinished business" — where the market established fair value but subsequently left without fully resolving. Price has a statistical tendency to return to these levels.

---

### 4.7 Auto S/R Level Extraction

Automatically extracts the top N most significant HVN levels and presents them as clean horizontal support/resistance lines.

| Aspect | Detail |
|---|---|
| **HVN Detection** | Local maxima in the volume distribution (volume row > both neighbors) |
| **Ranking** | HVNs ranked by total volume. Show top N (configurable, default: 5) |
| **Significance Filter** | Only show HVNs where volume exceeds a minimum threshold (e.g., > 1.5x median row volume) |
| **Dynamic Update** | S/R levels update as new volume data comes in |
| **Visual** | Thick horizontal lines at each HVN. Line thickness proportional to volume strength. |
| **Labels** | Price label on each S/R line with volume magnitude indicator (e.g., "S/R 55.40 ★★★") |

---

### 4.8 Dashboard Panel

A compact information panel displayed in the corner of the chart (similar to AKRTS).

| Row | Content |
|---|---|
| **Mode** | Current lookback mode and bar count |
| **POC** | Current POC price |
| **VAH / VAL** | Current Value Area boundaries |
| **Zone** | Current price position: "Above VAH", "Inside VA", "Below VAL" |
| **Nearest LVN** | Distance to nearest LVN zone (in ATR units) and direction |
| **Nearest nPOC** | Distance to nearest Naked POC (in ATR units) |
| **Imbalance** | Whether current price is in a Buy/Sell Imbalance zone |
| **Bias** | Derived directional bias based on volume structure: "Bullish" / "Bearish" / "Neutral" |

---

## 5. User Inputs (Settings Panel)

### 5.1 Volume Profile Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_vp_lookback_mode` | string | `"Fixed Bars"` | Fixed Bars / Session / N Sessions | How to define the profile range |
| `i_vp_lookback_bars` | int | `200` | 50–5000 | Number of bars (Fixed Bars mode) |
| `i_vp_num_sessions` | int | `1` | 1–10 | Number of sessions (N Sessions mode) |
| `i_vp_num_rows` | int | `50` | 24–200 | Number of price rows in the profile |
| `i_vp_value_area_pct` | float | `70.0` | 50.0–90.0 | Value Area percentage |
| `i_vp_show_histogram` | bool | `true` | — | Show/hide volume histogram |
| `i_vp_histogram_width` | int | `30` | 10–80 | Maximum histogram bar width (in bars) |

### 5.2 Key Level Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_show_poc` | bool | `true` | — | Show POC line |
| `i_show_vah_val` | bool | `true` | — | Show VAH/VAL lines |
| `i_poc_color` | color | `#FFEB3B` | — | POC line color (yellow) |
| `i_vah_color` | color | `#FF5252` | — | VAH line color (red) |
| `i_val_color` | color | `#4CAF50` | — | VAL line color (green) |
| `i_poc_style` | string | `"Solid"` | Solid / Dashed / Dotted | POC line style |
| `i_poc_width` | int | `2` | 1–4 | POC line width |

### 5.3 Volume Imbalance Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_imb_enabled` | bool | `true` | — | Enable Volume Imbalance detection |
| `i_imb_threshold` | float | `70.0` | 60.0–90.0 | Imbalance threshold percentage |
| `i_imb_min_vol_pct` | float | `50.0` | 20.0–80.0 | Minimum volume percentile for imbalance |
| `i_imb_buy_color` | color | `#00E676` | — | Buy imbalance zone color |
| `i_imb_sell_color` | color | `#FF1744` | — | Sell imbalance zone color |
| `i_imb_opacity` | int | `80` | 50–95 | Zone opacity (higher = more transparent) |

### 5.4 LVN Detection Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_lvn_enabled` | bool | `true` | — | Enable LVN zone detection |
| `i_lvn_percentile` | float | `20.0` | 10.0–40.0 | Volume percentile threshold for LVN |
| `i_lvn_color` | color | `#FF9800` | — | LVN zone color (orange) |
| `i_lvn_opacity` | int | `85` | 50–95 | LVN zone opacity |
| `i_lvn_alert` | bool | `true` | — | Alert when price enters LVN zone |

### 5.5 Multi-TF POC Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_mtf_enabled` | bool | `true` | — | Enable Multi-TF POC |
| `i_mtf_daily` | bool | `true` | — | Show Daily POC |
| `i_mtf_weekly` | bool | `true` | — | Show Weekly POC |
| `i_mtf_daily_color` | color | `#2196F3` | — | Daily POC color (blue) |
| `i_mtf_weekly_color` | color | `#9C27B0` | — | Weekly POC color (purple) |

### 5.6 Naked POC Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_npoc_enabled` | bool | `true` | — | Enable Naked POC tracking |
| `i_npoc_max_count` | int | `5` | 1–10 | Maximum Naked POCs to track |
| `i_npoc_color` | color | `#FF5722` | — | Naked POC line color |
| `i_npoc_alert_atr` | float | `0.5` | 0.1–2.0 | Alert when price within N ATR of nPOC |

### 5.7 Auto S/R Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_sr_enabled` | bool | `true` | — | Enable Auto S/R extraction |
| `i_sr_max_levels` | int | `5` | 3–10 | Maximum S/R levels to show |
| `i_sr_min_strength` | float | `1.5` | 1.0–3.0 | Minimum volume multiple above median |
| `i_sr_color` | color | `#78909C` | — | S/R line color |

### 5.8 Dashboard Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_show_panel` | bool | `true` | — | Show dashboard panel |
| `i_panel_position` | string | `"Top Right"` | Top Right / Top Left / Bottom Right / Bottom Left | Panel position |
| `i_panel_size` | string | `"Small"` | Small / Normal / Large | Panel text size |

### 5.9 Visual Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_buy_vol_color` | color | `#26A69A` | — | Buy volume histogram color (teal) |
| `i_sell_vol_color` | color | `#EF5350` | — | Sell volume histogram color (red) |
| `i_histogram_opacity` | int | `30` | 10–80 | Histogram bar opacity |

---

## 6. Alert System

| Alert ID | Condition | Message Template |
|---|---|---|
| `SVP_PRICE_AT_POC` | Price touches POC level | `"SVP: Price at POC ({{close}}) on {{ticker}}"` |
| `SVP_BREAK_VAH` | Price closes above VAH | `"SVP: Breakout above VAH ({{close}}) on {{ticker}}"` |
| `SVP_BREAK_VAL` | Price closes below VAL | `"SVP: Breakdown below VAL ({{close}}) on {{ticker}}"` |
| `SVP_ENTER_LVN` | Price enters an LVN zone | `"SVP: Price entering LVN zone — expect acceleration on {{ticker}}"` |
| `SVP_NEAR_NPOC` | Price within N ATR of Naked POC | `"SVP: Price approaching Naked POC at {{npoc_price}} on {{ticker}}"` |
| `SVP_BUY_IMBALANCE` | Price enters Buy Imbalance zone | `"SVP: Price at Buy Imbalance zone (institutional floor) on {{ticker}}"` |
| `SVP_SELL_IMBALANCE` | Price enters Sell Imbalance zone | `"SVP: Price at Sell Imbalance zone (institutional ceiling) on {{ticker}}"` |
| `SVP_POC_CONFLUENCE` | Multi-TF POC confluence detected | `"SVP: POC Confluence zone detected on {{ticker}}"` |

---

## 7. Performance Constraints

| Requirement | Target |
|---|---|
| **Non-Repainting** | All levels calculated from confirmed bar data. No `barmerge.lookahead_on`. |
| **Chart Load Time** | < 3 seconds on 5000-bar chart with default settings (50 rows, 200 bar lookback) |
| **Pine Compilation** | Must compile without warnings under Pine Script v6 strict mode |
| **Max Bars Back** | Explicitly managed via `max_bars_back` parameter |
| **Line/Box Limits** | Stay within TradingView's `max_lines_count` and `max_boxes_count` limits (500 each) |
| **Memory** | Efficient array usage; clean up expired boxes/lines to prevent memory leaks |
| **Mobile Rendering** | Dashboard and key levels must remain legible on TradingView mobile |

---

## 8. Integration with AKRTS

SVP is designed as a **companion indicator** to AKRTS. Together they provide:

| SVP Provides | AKRTS Provides | Combined Insight |
|---|---|---|
| **WHERE** (key price levels) | **WHEN** (entry/exit timing) | "Buy at this LVN breakout when AKRTS fires BUY signal" |
| Volume-based S/R | Kernel-based trend detection | "AKRTS says bullish + SVP shows Buy Imbalance below = high conviction long" |
| POC as target | Confluence score for entry | "Target the Daily POC when AKRTS partial exit fires" |
| LVN for acceleration zones | Mean Reversion for extremes | "Price at MR extreme + entering LVN = expect violent snap back" |

---

## 9. Out of Scope (v1)

- Real-time order flow / tape reading (requires Level 2 data not available in Pine)
- Time Price Opportunity (TPO) / Market Profile letters
- Footprint chart / bid-ask delta at tick level
- Volume Profile on custom drawing ranges (Pine Script limitation)
- Strategy mode with automated backtesting

---

## 10. Success Criteria

1. All 8 feature modules compile and render correctly on TradingView.
2. POC, VAH, VAL calculations match TradingView's built-in VRVP within ±1 tick tolerance.
3. LVN zones correctly identify acceleration areas on historical high-beta price action.
4. Volume Imbalance zones show statistically significant predictive value for S/R.
5. Naked POC levels demonstrate price magnet behavior on historical data.
6. Alert system fires correctly for all 8 alert conditions.
7. Dashboard panel provides clear, actionable real-time information.
8. Load time benchmark passes with default settings on high-beta stocks (SOXL, TQQQ).
