# Implementation decisions

Deviations from `PLAN.md` and resolutions of details the plan left open.
Each item: what changed, why, and what it means for the user.

---

## 1. Bias filter uses local 1H aggregation, not `request.security`

**Plan §2.3** proposed one `request.security` call for the 1H structure. The script
instead maintains a 1H candle stream from chart bars (the same aggregator used for the
CRT timeframes) and runs pivot detection over the completed 1H candles.

**Why:** deterministic and non-repainting, no security budget, no `lookahead` subtleties,
and the pivot history exists on the chart timeline anyway. Side effect: if the chart has
missing bars, the 1H structure can differ slightly from the native 1H chart.

## 2. SL risk is rounded *down* to whole ticks, not up

**Plan §7.8** said "round away from entry so realised RR ≥ configured", which contradicts
itself: rounding the stop away from the entry increases risk and *lowers* realised RR.
Implemented the stated invariant instead: risk is floored to whole ticks and clamped to
at least one tick, so `(TP − entry) / (entry − SL) ≥ RR` (except in the degenerate
sub-tick case, where the 1-tick minimum wins).

## 3. Same-bar evaluation on the arming bar

A setup is created on the first chart bar of C3 and is immediately evaluated against
that bar's high/low. Without this, entries in the first minute of C3 would be missed on
a 1m chart. Because chart TFs divide the CRT TFs, that bar belongs entirely to C3, so
no C2 price action leaks into the trigger.

## 3b. Same-bar outcome resolution on the trigger bar

The plan's rule ("if the bar's low ≤ SL on a long trigger bar → ambiguous → loss by
default") systematically mislabels trades: the trigger bar usually opens *below* the
long entry, so its low often predates the entry and dips below a tight SL. Implemented a
rule that only counts provably post-entry events on the trigger bar:

- **TP touched** → win. Reaching a TP beyond the entry requires crossing the entry, so
  the TP is necessarily post-entry (unless the bar opened beyond the entry *and* both
  TP and SL were touched, in which case the order is unknowable → `ambiguousOutcome`).
- **SL touched and the bar closed beyond it** → loss (the close is after the trigger).
- **SL wick only** → not resolved on that bar; tracking continues on the next bars.

From the bar after the trigger, the normal rule applies: both touched → `ambiguousOutcome`.

## 4. Loop order: aggregate → expire → step → arm

`PLAN.md` §3.4 stepped setups *before* aggregation. That would let an ARMED setup
trigger on the first bar *after* its trigger window, because the expiry signal only
appears at the HTF boundary. The implemented order is:

1. aggregate (detect HTF boundaries),
2. update the 1H bias,
3. expire setups whose window just ended,
4. step live setups with the current chart bar,
5. arm new setups on the boundary bar and step them immediately,
6. flush terminal setups and redraw.

## 5. Removed inputs: `keepDays` and `extendUntilResolved`

History is bounded by **Historical setups kept per timeframe** (`maxHistoryPerTf`,
default 15) which also bounds the drawing count. Lines always extend until resolution;
the option to not extend had no useful behaviour.

## 6. Debug reject markers are dashboard-only

**Plan §4** suggested `label.new` markers at rejected C2 closes. That can exhaust the
500-label budget on a busy chart. Reject reasons are shown in the dashboard row instead
(enable **Debug**). The detector's reason strings are unchanged.

## 7. Extra terminal states

- `SKIPPED` — the setup triggered but could not be taken: the `Triggered` slot budget
  was already consumed, or the entry filled at/beyond the planned TP (gap).
- `UNKNOWN` — TP and SL ambiguous per the `TP and SL in the same bar` input set to
  `Unknown`.

Both are drawn like other terminal states and included in the dashboard "Last" column.
`SKIPPED` setups are never alerted.

## 8. Session membership of C2

Evaluated from the **last chart bar of the C2 candle** (its `inSess` flag), not by
re-parsing the session string for an arbitrary timestamp. For a 1m chart the last bar of
a 13:00-close candle opens at 12:59, which is inside a `0930-1300` session.

## 9. TRIGGER / invalidation precedence

If a single chart bar both breaks the entry level and the C2 sweep extreme, the entry
wins (checked first). The bar is treated as C3 price action either way.

## 10. Entries are gated to the session (`triggerInSession`, default on)

The plan only filtered C2 closes by session, which allows a setup whose C2 closes exactly
at the session end to fill during the following candle (outside the window). Since the
strategy is a morning-session tool, the trigger is additionally required to occur while
the bar is inside the session. Turn it off to restore the plan's literal behaviour.

## 11. Fill price and gap handling

The fill is `max(open, entry)` for longs and `min(open, entry)` for shorts; TP/SL stay at
the planned levels. If the fill is already at/beyond TP, the setup is `SKIPPED` instead
of booking a nonsensical result.

## 12. Alert batching

Three `alert()` call sites (armed / triggered / result), each joined from a per-bar
buffer and using `alert.freq_once_per_bar`. This allows one alert of each type per bar;
the same type twice in one bar is merged, and a second event of the same type on the
same realtime bar is not re-sent. Static `alertcondition`s cover armed/triggered.

## 13. Trend-neutral handling

`neutralMode` applies only to the automatic bias modes (structure / EMA). `Manual: Both`
means "no filter" and never blocks.

## 14. Chart-timeframe guard

`runtime.error` on charts above 15m, and when an enabled CRT timeframe is not a multiple
of the chart timeframe (division-by-zero-safe nested checks). On a 15m chart with 15m
enabled, the dashboard shows a same-bar trigger warning instead of an error.

## 15. Object creation

One `Candle` object per completed HTF period per timeframe plus one per setup (two
references). Finalized candles are never mutated, so a setup's C1/C2 remain stable after
the context advances.

## 16. Chop filter: per-timeframe score, graded A–F

The request was a chop filter "for each timeframe" with a grading system: skip the
worst chop and only count sweeps in little/no chop. Implemented as an independent score
per CRT timeframe, computed from its own completed HTF candles (no `request.security`).

**Score (0–100):**

- 50% Chande chop index: `100 · log10(ΣTR(n) / (maxHigh(n) − minLow(n))) / log10(n)`,
  clamped to 0–100.
- 30% inverse Kaufman efficiency ratio: `(1 − |close_n − close_0| / Σ|Δclose|) · 100`.
- 20% average **body** overlap of consecutive candles: `bodyOverlap / min(body_i,
  body_i−1)`, averaged; a doji (zero body) counts as full overlap. Body overlap (not
  full-range overlap) keeps clean trends in grade A despite ordinary wicks.

**Grades:** A < 20, B < 35, C < 50, D < 65, F ≥ 65.

**Action tiers (inputs):**

- grade ≤ `chopCountMax` (default `B`): counted normally — consumes a slot and alerts.
- `chopCountMax` < grade < `chopSkipAt`: drawn as a ghost, no slot, no alerts. The
  ghost prefix names the reason (`chop C (moderate)`); `showChopGhosts` can hide it.
- grade ≥ `chopSkipAt` (default `F`): not drawn or tracked at all. `lastReject` becomes
  `chop F (extreme)`, visible in the dashboard's debug row.

The verdict is sampled at C2 close (same place as the bias decision) and stored on the
setup. It never changes while the setup is live, so a triggered trade is not abandoned
mid-flight. `n/a` (fewer than 3 completed HTF candles) counts as allowed. Counter-trend
and chop ghosts compose: both reasons appear in the label, and the setup is never
counted or alerted.

The default 20-candle lookback is 5h on 15m, 10h on 30m and 20h on 1H; the score is
deliberately faster on lower timeframes.
