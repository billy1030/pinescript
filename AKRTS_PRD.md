# PRD — Adaptive Kernel Regression Trading System (AKRTS)

## 1. Product Vision & Overview

**Indicator Name:** Adaptive Kernel Regression Trading System (AKRTS)  
**Platform:** TradingView  
**Language:** Pine Script v6  
**Type:** Overlay Indicator with optional companion oscillator  

AKRTS is a professional-grade, machine-learning-inspired overlay indicator that combines dual-kernel regression envelopes with multi-source dimensionality, first-pullback detection, tiered mean reversion signals, and a daily trend filter. It emits a standardized **-6 to +6 backtesting signal stream** for integration with external frameworks, while remaining visually clean, fully customizable, and highly performant for live auto-trading bots.

---

## 2. Target Users

| Persona | Needs |
|---|---|
| **Intraday Scalper / Day Trader (5m–15m)** | Mean-reversion scalping alerts, pullback entries, fast execution on volatile assets (e.g., SOXL). |
| **Discretionary Swing Trader** | Clean trend visualization on higher TF, but primarily utilizing 5m/15m charts. |
| **Algo / Bot Developer** | Non-repainting signal stream, webhook-ready alerts, low latency |
| **Quant Researcher** | Backtesting stream (-6 to +6), downsampling, remote fractals for pattern mining |
| **Accessibility-Conscious User** | Colorblind-safe palettes, high-contrast modes |

---

## 3. Feature Catalog

### 3.1 Dynamic Kernel Regression Envelope

A visual envelope derived from a **dual-kernel system** that communicates trend direction and provides confirmation via crossovers.

| Aspect | Detail |
|---|---|
| **Kernel A (Fast)** | Nadaraya-Watson estimator using a **Rational Quadratic (RQ)** kernel. Shorter lookback for responsiveness. |
| **Kernel B (Slow)** | Nadaraya-Watson estimator using a **Gaussian (RBF)** kernel. Longer lookback for stability. |
| **Envelope Bands** | Upper/Lower bands at ± N × ATR (or σ) from Kernel A. Configurable multiplier. |
| **Crossover Signal** | Kernel A crossing above/below Kernel B indicates trend shift (bullish/bearish). |
| **Fill Color** | Gradient fill between bands changes hue based on trend direction. |
| **Non-Repainting** | Endpoint estimation only — no future-bar look-ahead. |

**Kernel Math (Nadaraya-Watson):**

```
ŷ(x) = Σ K(x, xᵢ) · yᵢ  /  Σ K(x, xᵢ)
```

- **RQ Kernel:** `K(x, xᵢ) = (1 + |x − xᵢ|² / (2α·h²))^(−α)`
- **Gaussian Kernel:** `K(x, xᵢ) = exp(−|x − xᵢ|² / (2h²))`

---

### 3.2 First Pullback Detection System

Automatically identifies the **first pullback** after an initial momentum move for high-probability trend-continuation entries.

| Aspect | Detail |
|---|---|
| **Momentum Impulse** | Detected when price breaks above/below envelope band with volume confirmation. |
| **Pullback Zone** | Price retraces back toward Kernel A/B midline without violating the opposite envelope band. |
| **Continuation Trigger** | Price resumes the impulse direction — confirmed by a close beyond the pullback swing high/low. |
| **Visual Output** | **Green triangle (▲)** for bullish first pullback; **Red triangle (▼)** for bearish. |
| **State Machine** | Tracks states: `IDLE → IMPULSE_DETECTED → PULLBACK_IN_PROGRESS → CONTINUATION_TRIGGERED → COOLDOWN` |
| **Cooldown** | Configurable N-bar cooldown before the system resets for the next impulse cycle. |

---

### 3.3 Massively Expanded Dimensionality

Enables up to **15 total sources** (10 custom + 5 built-in) as features for the kernel regression distance calculation, allowing integration of virtually any TradingView indicator.

| Source Type | Count | Examples |
|---|---|---|
| **Custom External** | Up to 10 | Any `plot()` output from another indicator on the chart (RSI, MACD, OBV, custom scripts, etc.) |
| **Built-in** | 5 | `close`, `hlc3`, `hl2`, `ohlc4`, `volume` |

Each source receives a **configurable weight** (0.0–1.0) in the distance metric, allowing the user to emphasize or de-emphasize features.

**Distance Metric (Weighted Euclidean):**

```
d(x, xᵢ) = √( Σⱼ wⱼ · (xⱼ − xᵢⱼ)² )
```

Where `wⱼ` is the user-defined weight for source `j`.

---

### 3.4 Tiered Mean Reversion Signals with Scalping Alerts

Provides **two tiers** of mean-reversion signals based on statistical distance from the kernel regression line.

| Tier | Symbol | Condition | Use Case |
|---|---|---|---|
| **Regular** | ◆ (diamond) | Price exceeds **±1.5σ** from kernel midline | Moderate reversion opportunity |
| **Strong** | ◆/▼ or ◆/▲ | Price exceeds **±2.5σ** from kernel midline | High-confidence extreme reversion |

| Aspect | Detail |
|---|---|
| **σ Calculation** | Rolling standard deviation of the residuals (price − kernel estimate) over N bars. |
| **Confirmation Filter** | Optional RSI / Stochastic divergence confirmation to reduce false signals. |
| **Scalping Alerts** | Dedicated `alert()` calls for each tier: `MEAN_REV_REGULAR_LONG`, `MEAN_REV_REGULAR_SHORT`, `MEAN_REV_STRONG_LONG`, `MEAN_REV_STRONG_SHORT`. |
| **Visual** | Small shapes plotted at the signal bar. Color intensity differentiates tiers. |

---

### 3.5 Daily Kernel Trend Filter

A higher-timeframe kernel regression applied on the **Daily** chart to filter noisy intraday signals.

| Aspect | Detail |
|---|---|
| **Calculation** | Nadaraya-Watson (RQ kernel) computed on Daily close via `request.security()`. |
| **Trend State** | `BULLISH` if Daily kernel slope > 0 and price > kernel; `BEARISH` otherwise; `NEUTRAL` in between. |
| **Filter Effect** | When enabled, suppresses contra-trend First Pullback and Mean Reversion signals. |
| **Non-Repainting** | `lookahead = barmerge.lookahead_off` ensures no future data leakage. |
| **Visual** | Optional background tint (green/red/gray) indicating daily trend state. |

---

### 3.6 Professional Backtesting Stream (−6 to +6)

A **discrete integer signal** emitted every bar for external backtesting framework consumption.

| Value | Signal Meaning |
|---|---|
| **+6** | Strong Mean Reversion Long (extreme oversold) |
| **+5** | Regular Mean Reversion Long |
| **+4** | First Pullback Bullish Continuation |
| **+3** | Kernel A crosses above Kernel B (trend shift bullish) |
| **+2** | Price re-enters envelope from below (support bounce) |
| **+1** | Daily/Hourly trend filter turns bullish |
| **0** | Neutral / no signal |
| **−1** | Daily/Hourly trend filter turns bearish |
| **−2** | Price re-enters envelope from above (resistance rejection) |
| **−3** | Kernel A crosses below Kernel B (trend shift bearish) |
| **−4** | First Pullback Bearish Continuation |
| **−5** | Regular Mean Reversion Short |
| **−6** | Strong Mean Reversion Short (extreme overbought) |

- Emitted via `plot(signal, "Signal Stream")` for external tools.
- A corresponding `alertcondition()` fires on any non-zero value.

---

### 3.7 Zig-Zag (Swing High/Low) Tracker

A visual overlay that dynamically connects significant swing highs and lows, helping identify structural trend shifts and market geometry within the 5m-15m timeframe.

| Aspect | Detail |
|---|---|
| **Pivot Detection** | Identifies swing highs/lows based on a configurable `Pivot Lookback` (left and right bars). |
| **Line Rendering** | Draws a solid line (e.g., Blue) connecting alternating highest-highs and lowest-lows. |
| **Structural Context** | Helps visually confirm higher-highs (HH) and lower-lows (LL) during momentum impulses or kernel crossovers. |
| **Visual Toggle** | Can be enabled/disabled independently of the kernel envelope. |

---

### 3.8 Performance Optimizations

| Optimization | Detail |
|---|---|
| **Vectorized Distance Calc** | Pre-compute squared differences; avoid redundant `math.sqrt()` calls where only relative ordering matters. |
| **Lazy Kernel Evaluation** | Skip kernel weight computation for bars where `K(x, xᵢ) < ε` (negligible contribution). |
| **Cached HTF Data** | Store `request.security()` result in a variable; avoid repeated calls. |
| **Loop Unrolling** | For small fixed-size source arrays, manually unroll inner loops. |
| **Minimal Redraws** | Use `plot()` with `display = display.pane` for non-overlay outputs to reduce chart render cost. |
| **Max Bars Back** | Explicitly set `max_bars_back` on key series to prevent automatic (and expensive) bar buffer expansion. |

---

### 3.8 Full Visual Customization & Accessibility

| Category | Controls |
|---|---|
| **Color Theme** | Preset palettes: *Default*, *Dark Mode*, *Light Mode*, *Colorblind Safe (Deuteranopia, Protanopia, Tritanopia)*. |
| **Envelope Colors** | Bullish fill, bearish fill, neutral fill — each with opacity slider. |
| **Signal Markers** | Independent color + size for each signal type. |
| **Kernel Lines** | Style (solid/dashed/dotted), width, color for Kernel A and Kernel B. |
| **Background Tint** | On/Off + opacity for daily trend filter background. |
| **Label Visibility** | Toggle on/off for price labels, signal annotations, and debug info. |
| **Dark Mode Optimization** | Auto-detect or manual toggle; adjusts all default colors for dark backgrounds. |

---

### 3.9 Advanced Training Modes

Features for exotic pattern discovery and fine-tuning the kernel model.

| Mode | Detail |
|---|---|
| **Downsampling** | Evaluate the kernel regression on every Nth bar only, reducing noise and uncovering lower-frequency patterns. Configurable N (2–20). |
| **Remote Fractals** | Extend the lookback window to capture fractal structures at larger scales (e.g., 500–2000 bars back). Useful for identifying self-similar patterns across timeframes. |
| **Reset Factor** | A parameter (0.0–1.0) that controls how aggressively the neighbor pool is diversified. At 0.0, all historical bars are weighted equally; at 1.0, recent bars dominate entirely. Controls the "memory decay" of the kernel. |

---

## 4. User Inputs (Settings Panel)

### 4.1 Kernel Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_kernelA_type` | string | `"Rational Quadratic"` | RQ / Gaussian | Kernel A type |
| `i_kernelA_lookback` | int | `8` | 2–100 | Kernel A lookback window |
| `i_kernelA_bandwidth` | float | `8.0` | 0.1–50 | Kernel A bandwidth (h) |
| `i_kernelA_alpha` | float | `1.0` | 0.1–10 | RQ shape parameter α (Kernel A only) |
| `i_kernelB_type` | string | `"Gaussian"` | RQ / Gaussian | Kernel B type |
| `i_kernelB_lookback` | int | `24` | 2–200 | Kernel B lookback window |
| `i_kernelB_bandwidth` | float | `16.0` | 0.1–100 | Kernel B bandwidth (h) |
| `i_kernelB_alpha` | float | `1.0` | 0.1–10 | RQ shape parameter α (Kernel B only) |

### 4.2 Envelope Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_envelope_method` | string | `"ATR"` | ATR / StdDev | Band width method |
| `i_envelope_mult` | float | `2.0` | 0.5–5.0 | Multiplier for bands |
| `i_envelope_atr_len` | int | `14` | 5–50 | ATR length (if ATR method) |

### 4.3 First Pullback Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_pb_enabled` | bool | `true` | — | Enable/disable pullback detection |
| `i_pb_cooldown` | int | `10` | 3–50 | Bars before next impulse scan |
| `i_pb_vol_confirm` | bool | `true` | — | Require volume spike on impulse |
| `i_pb_vol_mult` | float | `1.5` | 1.0–3.0 | Volume spike multiplier (vs SMA vol) |

### 4.4 Mean Reversion Settings

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_mr_enabled` | bool | `true` | — | Enable mean reversion signals |
| `i_mr_regular_sigma` | float | `1.5` | 1.0–3.0 | Regular tier σ threshold |
| `i_mr_strong_sigma` | float | `2.5` | 2.0–4.0 | Strong tier σ threshold |
| `i_mr_residual_len` | int | `50` | 20–200 | Residual std dev lookback |
| `i_mr_rsi_confirm` | bool | `false` | — | RSI divergence confirmation |

### 4.5 Higher Timeframe (HTF) Trend Filter

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_htf_enabled` | bool | `true` | — | Enable HTF trend filter |
| `i_htf_res` | string | `"60"` | Any TF | HTF resolution (default 1-hour for 5m-15m charts) |
| `i_htf_lookback` | int | `20` | 5–100 | HTF kernel lookback |
| `i_htf_bandwidth` | float | `12.0` | 1.0–50 | HTF kernel bandwidth |
| `i_htf_show_bg` | bool | `true` | — | Show background tint |

### 4.6 Source Dimensionality

| Input | Type | Default | Description |
|---|---|---|---|
| `i_src_builtin_1` … `_5` | source | close, hlc3, hl2, ohlc4, volume | Built-in source selectors |
| `i_src_custom_1` … `_10` | source | close (placeholder) | External indicator source slots |
| `i_weight_builtin_1` … `_5` | float (0–1) | 1.0 | Weights for built-in sources |
| `i_weight_custom_1` … `_10` | float (0–1) | 0.0 | Weights for custom sources (0 = disabled) |

### 4.7 Backtesting Stream

| Input | Type | Default | Description |
|---|---|---|---|
| `i_bt_show_stream` | bool | `false` | Plot signal stream on separate pane |
| `i_bt_alert_any` | bool | `true` | Fire alert on any non-zero signal |

### 4.9 Advanced Training

| Input | Type | Default | Range | Description |
|---|---|---|---|---|
| `i_adv_downsample` | int | `1` | 1–20 | Downsample factor (1 = off) |
| `i_adv_remote_frac` | bool | `false` | — | Enable remote fractals |
| `i_adv_remote_lookback` | int | `500` | 200–2000 | Remote fractal lookback |
| `i_adv_reset_factor` | float | `0.5` | 0.0–1.0 | Neighbor diversity reset factor |

### 4.10 Visual / Accessibility

| Input | Type | Default | Description |
|---|---|---|---|
| `i_theme` | string | `"Dark Mode"` | Color theme preset |
| `i_bull_color` | color | `#00E676` | Bullish color |
| `i_bear_color` | color | `#FF1744` | Bearish color |
| `i_neutral_color` | color | `#78909C` | Neutral color |
| `i_kernel_a_color` | color | `#448AFF` | Kernel A line color |
| `i_kernel_b_color` | color | `#FFAB40` | Kernel B line color |
| `i_kernel_a_width` | int | `2` | Kernel A line width |
| `i_kernel_b_width` | int | `1` | Kernel B line width |
| `i_fill_opacity` | int | `15` | Envelope fill opacity (0–100) |
| `i_show_zigzag` | bool | `true` | Show Swing High/Low Zig-Zag line |
| `i_zigzag_len` | int | `5` | Pivot lookback for Zig-Zag |
| `i_zigzag_color` | color | `#2196F3` | Zig-Zag line color (Blue) |
| `i_show_labels` | bool | `true` | Show signal labels |
| `i_show_debug` | bool | `false` | Show debug info table |

---

## 5. Alert System

| Alert ID | Condition | Message Template |
|---|---|---|
| `AKRTS_BULL_PULLBACK` | First pullback bullish triggered | `"AKRTS: Bullish First Pullback on {{ticker}} ({{interval}})"` |
| `AKRTS_BEAR_PULLBACK` | First pullback bearish triggered | `"AKRTS: Bearish First Pullback on {{ticker}} ({{interval}})"` |
| `AKRTS_MR_REG_LONG` | Regular mean reversion long | `"AKRTS: Regular Mean Reversion Long on {{ticker}}"` |
| `AKRTS_MR_REG_SHORT` | Regular mean reversion short | `"AKRTS: Regular Mean Reversion Short on {{ticker}}"` |
| `AKRTS_MR_STR_LONG` | Strong mean reversion long | `"AKRTS: STRONG Mean Reversion Long on {{ticker}}"` |
| `AKRTS_MR_STR_SHORT` | Strong mean reversion short | `"AKRTS: STRONG Mean Reversion Short on {{ticker}}"` |
| `AKRTS_TREND_BULL` | Kernel A crosses above B | `"AKRTS: Bullish Trend Shift on {{ticker}}"` |
| `AKRTS_TREND_BEAR` | Kernel A crosses below B | `"AKRTS: Bearish Trend Shift on {{ticker}}"` |
| `AKRTS_DAILY_BULL` | Daily filter turns bullish | `"AKRTS: Daily Trend Now Bullish on {{ticker}}"` |
| `AKRTS_DAILY_BEAR` | Daily filter turns bearish | `"AKRTS: Daily Trend Now Bearish on {{ticker}}"` |
| `AKRTS_SIGNAL_STREAM` | Any non-zero signal value | `"AKRTS Signal: {{plot_0}} on {{ticker}} ({{interval}})"` |

---

## 6. Non-Functional Requirements

| Requirement | Target |
|---|---|
| **Non-Repainting** | All signals must use endpoint estimation only. No `barmerge.lookahead_on`. |
| **Chart Load Time** | < 2 seconds on 5000-bar chart with default settings. |
| **Pine Compilation** | Must compile without warnings under Pine Script v6 strict mode. |
| **Max Bars Back** | Explicitly managed; no runtime errors on low-bar-count symbols. |
| **Webhook Compatibility** | Alert messages must be parseable JSON or clean strings for webhook endpoints. |
| **Mobile Rendering** | All visuals must remain legible on TradingView mobile app. |

---

## 7. Out of Scope (v1)

- Full strategy (`strategy()`) mode with automatic backtesting — indicator only.
- Order execution / broker integration.
- Multi-symbol scanner dashboard.
- AI/ML model training outside of PineScript (e.g., Python integration).

---

## 8. Success Criteria

1. All 9 feature modules compile and render correctly on TradingView.
2. Signal stream values match documented mapping for every signal type.
3. Daily trend filter provably reduces contra-trend false signals in visual comparison.
4. Alert system fires correctly for all 11 alert conditions.
5. Load time benchmark passes on default settings.
6. Colorblind-safe palette passes WCAG AA contrast ratio checks.
