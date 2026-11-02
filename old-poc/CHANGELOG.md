# Changelog

All notable changes to the CRT Nitro indicator. Semantic versioning; every
release is a pre-release until the user marks the verification checklist
complete (`crt-nitro-indicator-verification-tests.md`).

## [0.9.0-alpha.1] — 2026-10-13 — Prop-eval risk governor

- Governor inputs: account size 50k, profit target 3k, max drawdown 2k,
  drawdown type (EOD default / intraday), daily loss limit 300, floor mode
  (trailing EOD peak / static), optional stop-arming at the profit target.
- Reference equity ledger seeded at the account size; daily-loss lockout
  (day P&L ≤ −DLL) blocks and cancels new candidates for the rest of the
  trading day while active references keep being monitored; lockout takes
  precedence over directional quotas and survives session boundaries.
- Account floor = max(peak EOD reference equity − DD, size − DD); signals whose
  reserved risk would breach the floor are blocked (`GOV_FLOOR_BLOCKED`).
- Reserved-risk budget default drops to $300 (Q3) and is clamped to the DLL;
  admission budget = min(DLL headroom, floor headroom, clamped budget).
- Governor dashboard block and data-window series; all cells labelled
  "reference accounting; NOT broker P&L".

## [0.8.0-alpha.1] — 2026-10-13 — Outcome ledger and lifecycle alerts

- Per-day, per-session and cumulative tallies: target, stop, ambiguous,
  expired, session-end, day-end.
- Reference-only P&L: frozen qty × points × point value minus the cost
  allowance per resolved side-pair; ambiguous outcomes are counted as losses
  and shown separately; unresolved endings carry zero P&L.
- `alert()` on every terminal lifecycle event; separate `alertcondition`s for
  target-reached and stop/ambiguous per side.
- Data-window plots for every tally; cumulative values depend on loaded history.

## [0.7.0-alpha.1] — 2026-10-13 — 1H bias filter

- Restores the original indicator's 1H bias filter: sign of the last confirmed
  60-minute candle body, `lookahead_off` with a `[1]` offset (never a
  developing HTF value); on a 60m chart it uses the chart's own prior candle.
- Modes: Require alignment (default; flat bias blocks both) or Block
  counter-bias only (flat bias allows both). Filter OFF by default.
- `BIAS_BLOCKED` at admission; pending candidates are cancelled if bias turns
  counter. Dashboard cells for bias direction, candle close and blocking state.

## [0.6.0-alpha.1] — 2026-10-13 — Directional concurrency

- Per-direction reference slots (slot 0 = LONG, slot 1 = SHORT); a LONG and a
  SHORT reference may be active simultaneously. "Maximum concurrent
  references" 1 (single-reference behaviour) or 2 (default).
- Per-direction lifecycle evaluation, same-candle lock and drawings; one
  entry/stop/target line and label trio per direction; drawing limits raised.
- 5m charts are chop-diagnostics only; the CRT engine requires 15m+ (Q1).
- Dashboard gains per-slot reference rows and a max-concurrent cell.

## [0.5.1-alpha.1] — 2026-10-13 — Signal integrity and throughput

- D-01: a candle that terminates a pending candidate (expiry, cancellation,
  invalidation, target-consumed) can arm a fresh candidate on that same
  candle; TRIGGERED still blocks.
- D-02 (parts 1/3): session boundary mode Snap to chart candles (default) /
  Strict; NY default widened to 0930-1600 RTH so every supported timeframe has
  usable session candles; "Eligible candles / session" dashboard cell.
- D-03: entry window default raised to 3 candles; Auto-mode threshold is an
  input; Auto falls back to Direct (with a note) instead of silently rejecting.
- D-04: candidate levels commit only on WAIT_BREAK — rejected sweeps no longer
  repaint the candidate dashboard block.
- D-05: single `CHOP_A..CHOP_D` boundary constants; "Minimum chop grade for
  entry" input (A / B, default B).
- D-06: session strings validated (shape, ranges, optional day mask); an
  invalid string surfaces "Invalid session string" instead of arbitrary keys.
- D-07: chop score denominators match actual sample counts.
- D-08: dead state removed (`crtLastPlanEndBar`), guard branch documented,
  chat-chunk markers replaced by section banners.
- D-09: trading day rendered in the exchange timezone.
- D-10: duplicate issues file removed.

## [0.5.0-alpha.1] — earlier session — Reference lifecycle

- Single-active-reference lifecycle: post-entry target/stop observation,
  ambiguous (both-levels) classification, candle/session/day expiry, no risk
  refund; entry alerts identify the version.

## [0.4.0-alpha.1] — earlier session — Estimated risk model

- Micro-equivalent sizing (MNQ/MES/MYM/M2K), max risk per signal, daily
  reserved-risk budget, max contracts, minimum gross R:R, cost allowance and a
  combined daily cap; stops stay structural.

## [0.3.0 / 0.3.1-alpha.1] — earlier session — Counters, plans, alerts

- Separate daily long/short counters, frozen plan levels + grade, entry/stop/
  target drawings, long/short alerts.

## [0.2.0-alpha.1] — earlier session — Native-timeframe CRT engine

- Confirmed C1/C2 detection, Direct vs Re-break sequencing, C3-only
  expiration, session filter, structural invalidation, diagnostic markers.

## [0.1.0 / 0.1.1-alpha.1] — earlier session — Chop diagnostics

- Closed-candle chop score, sampled history, Improving/Worsening/Constant,
  supported-timeframe guard, current-timeframe-only dashboard.
