# CRT Nitro — Sweep Setups (Pine Script v6)

Indicator for the Candle Range Theory (CRT) / liquidity-sweep sequence described in
`vid_crt_one_perfect_trade_per_day.txt`. Built for ES / NQ / MES / MNQ, works on any symbol.

It detects the C1→C2→C3 sequence on **4H, 1H and 30m simultaneously**, grades the
chop level of each timeframe independently, and draws the entry / TP / SL only for
setups that actually **trigger** (first 2 setups of the session per timeframe). It
tracks each trade to its outcome and fires alerts.

---

## Install

1. Open TradingView → Pine Editor → paste `crt_nitro.pine` → *Add to chart*.
2. Use a **1m chart** (recommended), or 2m / 3m / 5m / 15m. Every chart timeframe up
   to 15m divides the 30m / 1H / 4H CRT timeframes.
3. Symbol: `ES1!`, `NQ1!`, `MES1!`, `MNQ1!` (continuous or the current contract).

The script **refuses to run** if:
- the chart is not time-based,
- the chart timeframe is above 15m, or
- an enabled CRT timeframe is not an integer multiple of the chart timeframe.

---

## What it draws

Only setups that actually trigger are drawn. Armed, expired and invalidated setups
leave no chart markings. Every triggered setup shows:

| Element | Meaning |
|---|---|
| Solid line in the TF color (teal 4H / purple 1H / orange 30m) | **Entry** = C2 extreme on the non-swept side |
| Teal solid line | **TP** = opposite extreme of C1 |
| Red dashed line | **SL** = entry −/+ (reward / RR) |
| Faint TF-colored box | C1 range (high/low **including wicks**) |
| Tiny triangle at the C2 wick | Where the sweep happened |
| Right-hand label | `TF ▲/▼ #n · chop grade · R · contracts · result` |
| Green/red boxes after trigger | Reward and risk zones |

Label result suffixes: `open`, `✔ TP`, `✘ SL`, `? AMBIGUOUS`. Filtered ghosts
(counter-trend and/or grades between the count and skip thresholds) are gray and
dotted with a `✕ filtered` prefix; they are hidden by default.

Lines extend bar by bar until the setup resolves, then freeze. Historical drawings are
kept per timeframe (drawing budget).

---

## Rules implemented

On each enabled timeframe, when a candle closes:

1. **C1** = the candle before it (range incl. wicks).
2. **C2** = the candle that just closed:
   - wick breaks **exactly one** side of C1 (`high > C1.high` XOR `low < C1.low`),
   - close is back inside `(C1.low, C1.high)`.
3. The setup arms **only if C2 closed inside the session**, the slot budget is free,
   and the timeframe's chop grade allows it (see below).
4. **Entry**: first chart bar (1m) that trades through the C2 extreme on the non-swept
   side, within the trigger window (default: C3 only).
5. **TP** = the opposite C1 extreme. **SL** is derived so `reward / risk = RR`.
6. Outcome is resolved by chart-bar touches (`>=` TP / `<=` SL for longs).
7. A setup with no entry by the end of the window becomes `EXPIRED`; if price breaks
   the sweep extreme before entry, it becomes `INVALIDATED`.

Direction filter: a bullish CRT (low swept) is only taken with a bullish 1H bias,
and vice versa. Neutral bias allows both by default.

**Chop filter (per timeframe).** Each timeframe is scored independently from its
completed HTF candles. The grade is sampled when C2 closes:

| Grade | Score | Meaning | Default action |
|---|---|---|---|
| A | 0–20 | little/no chop | counted |
| B | 20–35 | low chop | counted |
| C | 35–50 | moderate chop | skipped (not drawn) |
| D | 50–65 | high chop | skipped (not drawn) |
| F | >= 65 | extreme chop | skipped (not drawn) |

"Count" means the setup consumes a slot, fires armed/triggered/result alerts, and is
tracked normally. Skipped setups are not drawn or tracked at all. The thresholds are
inputs; grades between "Count up to" and "Skip from" become faded ghosts when
**Draw filtered setups faded** is enabled. With the defaults, only A/B sweeps are
shown — high-chop (C–F) environments leave the chart clean.

The score combines the Chande chop index (50%), inverse Kaufman efficiency ratio (30%)
and consecutive-candle body overlap (20%). `0` = strongly directional, `100` = pure
chop. A lookback of 20 HTF candles means 10 hours on 30m, 20 hours on 1H and 80 hours
on 4H. The grade is computed from completed candles only (no repainting).

---

## Inputs

### Session
- **Session** — default `0930-1300` (video). Set `0800-1300` for your window.
- **Timezone** — default `America/New_York`; the session string is interpreted in it.
- **Max setups per timeframe per session** — default 2.
- **Consume a slot on** — `Armed` (every detected setup counts) or `Triggered`
  (only setups that actually trigger count).

### Timeframes
- Toggles for 4H / 1H / 30m.
- **Trigger window** — number of HTF candles after C2 in which the entry is valid
  (1 = C3 only, the video rule).

### Trade
- **Risk:Reward** — default 1.5.
- **Minimum reward / minimum stop (ticks)** — 0 = off. Use e.g. 8 reward ticks to
  skip micro-range setups.
- **Invalidate if the C2 sweep extreme breaks before entry** — default on.
- **Only trigger entries inside the session** — default on, so a setup whose C2 closes
  at the session boundary cannot fill after your window.
- **TP and SL in the same bar** — `Loss` (default, conservative), `Win`, or `Unknown`.
- **Require C2 open inside the C1 range** — stricter body test.

### Chop filter
- **Enable chop filter** — default on. Scores each CRT timeframe separately.
- **Chop lookback (HTF candles)** — default 20 completed candles per timeframe
  (10h / 20h / 80h, respectively).
- **Count setups up to grade** — default `B` (low chop). `A`/`B` count; higher grades
  are skipped unless ghosting is enabled.
- **Skip setups from grade** — default `C` (moderate). C–F are neither drawn nor
  tracked.
- **Draw filtered setups faded** — default off. On shows grades between the count and
  skip thresholds as gray ghosts (never counted or alerted).

### Bias filter
- **Auto 1H structure** (default): compares the last two confirmed 1H swing highs and
  lows — HH+HL = bull, LH+LL = bear, otherwise neutral.
- **EMA 1H**: price above/below a 1H EMA.
- **Manual**: Long / Short / Both.
- **When the 1H bias is neutral**: allow Both (default) or None.
- **Draw counter-trend setups faded** — shown but never counted or alerted.

### Sizing
- **Account risk ($)** — if > 0, the label appends the contract count:
  `floor(riskUsd / (stopTicks × tickValue))`, where `tickValue = mintick × pointvalue`.

### Visuals
- Colors per timeframe, C1 box, reward/risk boxes, sweep markers.
- Historical setups kept per timeframe (drawing budget).
- **Setup label font size** and **Dashboard font size** — Tiny / Small / Normal / Large / Huge.
- Dashboard position, debug reject reasons.

### Alerts
- Armed / Triggered / Result toggles. Messages include symbol, TF, direction, slot,
  entry, TP, SL, R and the chop grade/score. Multiple events on one bar are joined
  with newlines.

---

## Dashboard

`1H bias` · `Session OPEN/CLOSED` · per-TF row: `slots used`, `active setup state`,
`chop grade + score` (green = count, yellow = faded, red = skipped), `last result`.
The chop row shows the active count/skip thresholds and lookback. With **Debug** on,
the last reject reason per timeframe is shown (`no sweep`, `both sides swept`,
`C2 closed outside range`, `counter-trend`, `reward below minimum`, `chop F
(extreme)`, ...).

---

## Alerts setup

Add one alert on the indicator with *Any alert() function call* for the rich messages,
or use the built-in **CRT armed** / **CRT triggered** conditions for simple alerts.

---

## Known limitations

- **HTF candles are built locally from chart bars**, not fetched with `request.security`.
  On symbols/timeframes where the chart has missing bars (rare on CME futures), the
  constructed HTF candle can differ slightly from TradingView's native candle.
- **Intrabar order is unknown.** On the **trigger bar** only provably post-entry events
  count: TP touched = win (reaching TP beyond the entry implies crossing the entry),
  close beyond SL = loss; an SL wick alone is ignored because it may predate the entry.
  If the bar opened beyond the entry and both TP and SL were touched, the configured
  `ambiguousOutcome` is used. From the next bar on, TP and SL touched in one bar uses
  `ambiguousOutcome` (default: loss).
- **Bias pivots lag by `pivotLen` HTF bars** (inherent to confirmed pivots). The 1H
  structure uses completed candles only — no repainting.
- The 4H timeframe needs 4 hours of in-session HTF aggregation before its first C2
  can close; use a lower chart timeframe and leave it running.
- The chop grade is a heuristic, sampled when C2 closes and never re-evaluated while
  the setup is live. It filters entries, not exits: a trade already triggered is
  tracked to TP/SL even if the regime changes.
- The score needs 3 completed HTF candles; before that the grade is `n/a` and setups
  are allowed. With the default lookback the score reflects 10h/20h/80h of data, so it
  adapts faster on 30m than on 4H.
- Alerts fire once per bar per event type. Two *different* events of the same type in
  one bar are merged into one message; a second event of the same type later in the
  same realtime bar is not re-sent.

---

## Validation checklist

Since TradingView is the only Pine runtime, verify after pasting:

1. **Video example** — MNQ, 2026-05-05. In bar replay on 1m, the first in-session
   sweep candle that closes back inside its previous candle arms; the setup is drawn
   only once entry triggers; the trade resolves.
2. **Geometry** — on any triggered setup, `(TP − entry) / (entry − SL)` must be >=
   the configured RR (SL risk is rounded down to whole ticks).
3. **Counting** — no timeframe shows more than 2 counted setups per session; the
   dashboard slot counter matches the drawn labels; counters reset at the next session.
4. **Expiry** — a setup that never trades through C2 leaves no drawing at the next HTF
   boundary.
5. **Bias** — switch to Manual Long/Short and confirm the dashboard bias row and that
   counter-trend setups are hidden (or ghosted when enabled).
6. **Performance** — 1m chart, 20k bars, all TFs on, debug off: the script should load
   without a timeout.
7. **Realtime** — leave it running through an HTF close and confirm one alert per bar.
8. **Chop** — compare the per-TF dashboard scores on a trending day and a ranging day.
   With the defaults, only A/B setups are tracked and drawn; C–F leave the chart clean
   (debug shows `chop C (moderate)` etc.). Alerts for counted setups include
   `CHOP <grade> <score>`.
