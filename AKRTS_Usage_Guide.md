# AKRTS — User Guide & Color Reference

## Quick Start
1. Open TradingView → Pine Editor → paste the contents of `AKRTS.pine`
2. Click **Add to Chart**
3. Use on a **5m or 15m** chart (optimized for SOXL/SOXS and leveraged ETFs)

---

## Color & Visual Reference

### Background Tint (HTF Trend Filter)
| Color | Meaning |
|---|---|
| **Light Blue background** | The 1-hour (HTF) trend is **BULLISH** — favor long trades |
| **Light Yellow background** | The 1-hour (HTF) trend is **BEARISH** — favor short trades |
| **No tint (clear)** | HTF trend is **NEUTRAL** — no directional bias |

### Kernel Lines
| Line | Default Color | Meaning |
|---|---|---|
| **Kernel A (Fast)** | Blue (#448AFF) | Responsive trend line — reacts quickly to price |
| **Kernel B (Slow)** | Orange (#FFAB40) | Stable trend line — smooth, slower to move |
| **Kernel A above Kernel B** | Envelope fills **GREEN** | Overall trend is **BULLISH** |
| **Kernel A below Kernel B** | Envelope fills **RED** | Overall trend is **BEARISH** |

### Envelope Bands
| Element | Color | Meaning |
|---|---|---|
| **Upper Band** | Faint red line | Resistance / overbought zone boundary |
| **Lower Band** | Faint green line | Support / oversold zone boundary |
| **Fill between bands** | Green or Red (semi-transparent) | Trend direction — green = bull, red = bear |

### Signal Shapes on Chart
| Shape | Color | Size | Signal |
|---|---|---|---|
| ▲ Triangle Up | **Green** | Normal | **Bullish First Pullback** — trend continuation entry |
| ▼ Triangle Down | **Red** | Normal | **Bearish First Pullback** — trend continuation entry |
| ◆ Diamond | **Green** (faded) | Tiny | **Regular Mean Reversion Long** — price at -1.5σ |
| ◆ Diamond | **Green** (bright) | Normal | **STRONG Mean Reversion Long** — price at -2.5σ (high probability) |
| ◆ Diamond | **Red** (faded) | Tiny | **Regular Mean Reversion Short** — price at +1.5σ |
| ◆ Diamond | **Red** (bright) | Normal | **STRONG Mean Reversion Short** — price at +2.5σ (high probability) |

### Deep Dive: The Two Trading Strategies

These two shapes represent **two completely different ways to make money**:

#### 1. Green/Red Triangles: [Trend Continuation] (Pullback)
**This is the safest, highest-win-rate approach, perfect for steady traders.**
* **The Logic:** The trend has already started (the bus left the station), but it always takes breaks (price pullbacks). The triangle tells you: "The bus paused, jump on now!"
* **▲ Green Triangle (Long):** During a big uptrend, the price dips briefly to rest, then readies to surge again. Buy here.
* **▼ Red Triangle (Short):** During a big downtrend, the price bounces up briefly to catch its breath, then readies to plummet again. Sell here.
* **💡 The Ultimate Combo:**
  * **Light Yellow Background (Bullish)** + **Green Triangle** = **Perfect Buy Setup! High Win Rate!**
  * **Light Blue Background (Bearish)** + **Red Triangle** = **Perfect Sell Setup! High Win Rate!**

#### 2. Green/Red Diamonds: [Extreme Reversal] (Mean Reversion)
**This is the "catch the falling knife" approach, meant for aggressive traders hunting absolute tops and bottoms.**
* **The Logic:** Price acts like a rubber band. If stretched too far (overbought/oversold), it will snap back violently. The diamond warns: "The band is about to snap!"
* **◆ Faded Small Diamond (Minor Stretch):**
  * The rubber band is slightly stretched. It is not recommended to enter new trades here. Instead, treat it as a **"Warning Signal"** to take partial profits on existing trades.
* **◆ Bright Large Diamond (Extreme Stretch, High Reversal Probability):**
  * **Bright Green Large Diamond (Oversold):** Market panicked and sold off too hard. Expect a violent bounce upwards. Good place for a speculative **Long**.
  * **Bright Red Large Diamond (Overbought):** Market is overly euphoric and bought too much. Expect a sudden drop. Good place for a speculative **Short**.
* **💡 Strategy Tip:** Trading large diamonds is high-risk, high-reward. You can trade them even against the background color, but secure profits quickly and use strict stop losses!

### Buy/Sell/Exit Labels
| Label | Color | Meaning |
|---|---|---|
| **B1 / 50%** | Green ▲ | **Tier 1 Buy** — open 50% position (confluence ≥ 5) |
| **B2 / +25%** | Green ▲ (small) | **Tier 2 Buy** — add 25% to position (confluence ≥ 7) |
| **B3 / +25%** | Green ▲ (small) | **Tier 3 Buy** — final 25% add-on (confluence ≥ 9, total = 100%) |
| **S1 / 50%** | Red ▼ | **Tier 1 Sell** — open 50% short (confluence ≥ 5) |
| **S2 / +25%** | Red ▼ (small) | **Tier 2 Sell** — add 25% short (confluence ≥ 7) |
| **S3 / +25%** | Red ▼ (small) | **Tier 3 Sell** — final 25% short (confluence ≥ 9) |
| **SL ✕** | Orange | **Stop Loss** — 100% position closed |
| **TP ✕** | Green | **Take Profit** — 100% position closed |
| **Trail ✕** | Orange | **Trailing Stop** hit — 100% position closed |
| **Cross ✕** | Orange | Kernel crossover against position — 100% closed |
| **Partial XX%** | Light Orange | Partial exit — 25% or 50% of position closed |

### Zig-Zag Line
| Element | Color | Meaning |
|---|---|---|
| **Dotted Blue diagonal lines** | Blue (#2196F3) | Connects swing highs and lows — shows market structure |

### Position Panel (Top-Right)
| Field | Shows |
|---|---|
| **Position** | LONG / SHORT / FLAT (color-coded) |
| **Size** | Current position size as % (0%, 50%, 75%, 100%) |
| **Entry** | Entry price of current position |
| **P&L** | Unrealized profit/loss (green = profit, red = loss) |
| **Conf L/S** | Current confluence score for Long / Short |

---

## How to Read the Signals

### The Decision Flow
1. **Check Background** → Light Blue = HTF bullish bias, Light Yellow = HTF bearish bias
2. **Check Kernel Lines** → Blue above Orange = uptrend, below = downtrend
3. **Watch for Shapes** → Triangles = pullback entries, Diamonds = mean reversion
4. **Watch for Labels** → B1/B2/B3 = tiered buys, S1/S2/S3 = tiered sells

### Example Trade (Long on SOXL 5m chart)
1. Background turns **light blue** (HTF 1-hour trend is bullish)
2. Kernel A (blue) crosses **above** Kernel B (orange) — trend shift confirmed
3. Price breaks above upper band with volume → impulse detected
4. Price pulls back inside the envelope → pullback in progress
5. Price breaks above the impulse high → **Green ▲ triangle** appears
6. Confluence score hits 5+ → **B1 (50%)** label appears — enter 50%
7. Score climbs to 7+ → **B2 (+25%)** label — add 25%
8. Price reaches kernel midline on a dip → **Partial 25%** label — take some off
9. Trailing stop catches the rest → **Trail ✕** label — fully closed

---

## Confluence Scoring Breakdown
The system scores each bar from 0 to 15 for both long and short directions:

| Factor | Points |
|---|---|
| HTF trend alignment | +2 |
| Kernel A on correct side of B | +2 |
| First Pullback signal fires | +3 |
| Regular Mean Reversion signal | +2 |
| Strong Mean Reversion signal | +4 |
| Price inside envelope bands | +1 |
| Volume above average | +1 |
| **Maximum possible** | **15** |

### Tier Thresholds (Adjustable in Settings)
- **Tier 1 (50% entry):** Score ≥ 5 (default)
- **Tier 2 (+25% add):** Score ≥ 7 (default)
- **Tier 3 (+25% max):** Score ≥ 9 (default)

---

## Trading in the White (Neutral) Zone

When the background is **white (clear)**, the higher timeframe (1-Hour) is telling you: **"The big trend is currently confused or resting. There is no clear wind pushing the market up or down."**

Here is exactly what you should do during those white periods:

### 1. Wait for the "Cross" Signal 
When the blue line crosses below the orange line (or vice versa), the short-term trend has officially shifted. This crossover creates an orange **"Cross ✕"** label.
* **What to do:** This is your signal to start looking for opportunities in the direction of the cross, even if the background hasn't fully turned color yet.

### 2. Take Quick Scalps (Hit and Run)
Because there is no strong 1-hour trend supporting you, you should not expect massive, long-lasting moves.
* If a triangle (Pullback) or diamond (Mean Reversion) signal appears, you can still take the trade.
* **What to do:** Take your profits much faster than usual. Don't wait for a huge home run. As soon as you are in profit, move your stop loss to break-even or take partial profits quickly.

### 3. Let the Confluence System Do the Math
The AKRTS Position Manager handles this math for you. Because the background is white, the system **will not** give you the +2 bonus confluence points for HTF alignment.
* **What to do:** If the indicator still prints a **"B1"** or **"S1"** label in the white zone, it means the short-term setup is so strong that it outweighs the lack of a 1-hour trend. You can take the trade, but keep your position size strictly to what the label says.
