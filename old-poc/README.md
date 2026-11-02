# CRT Nitro Indicator (Pine Script v6)

A TradingView **indicator** (not a strategy) implementing a Candle Range Theory
(CRT) reference engine: chop qualification, native-timeframe C1/C2 sweeps,
sequential entry state machine, per-direction reference lifecycle, an outcome
ledger with reference-only P&L, and a prop-eval risk governor.

Current version: **0.9.0-alpha.1 — pre-release / unstable.**
Verification is incomplete by design; see the checklist. Promotion to
`1.0.0` is user-gated.

## Files

| File | Purpose |
| --- | --- |
| `crt-nitro-indicator.pine` | The indicator; paste into the TradingView Pine Editor |
| `crt-nitro-indicator-verification-tests.md` | Living verification checklist (status per row) |
| `AGENTS.md` | Working state for anyone resuming development |
| `CHANGELOG.md` | Version history |
| `crt-nitro-indicator-build-plan.md` | Original completion plan / defect register |
| `crt-nitro-indicator-reqs.md` | Original requirements |

## Install

1. TradingView → Pine Editor → paste `crt-nitro-indicator.pine` → Save.
2. Add to chart. Use a **standard** 15m / 30m / 60m chart (5m charts show chop
   diagnostics only; other timeframes show a guard message).
3. For alerts, create them for the `alert()` calls ("Any alert() function
   call") and/or the `alertcondition` entries, and recreate them after any
   settings change.

## Supported charts and instruments

- Standard time-based candles; 15m / 30m / 60m for the CRT engine; 5m for chop
  diagnostics.
- Sized for ES/MES, NQ/MNQ, YM/MYM, RTY/M2K USD futures (micro-equivalent
  sizing by default: full-size charts, micro risk). Micro point values:
  MNQ 2.0, MES 5.0, MYM 0.5, M2K 5.0 USD per point.
- Non-futures or non-USD symbols fail the risk model with `RISK_BLOCKED`.

## What it does

- **Chop module** — closed-candle score (alternating bodies, wicks, alternating
  sweeps), A–F grades, sampled history, Improving/Worsening/Constant.
- **CRT engine** — one-sided C2 sweep of C1, confirmed close beyond C2's
  extreme, Direct or Re-break sequencing, structural stop, C1 target, R:R and
  risk admission, per-direction daily quotas.
- **References** — up to one LONG and one SHORT reference concurrently;
  frozen levels/grade/size; post-entry observation (target/stop/ambiguous/
  expiry/session/day). Ending a reference never refunds reserved risk.
- **1H bias filter** — optional; sign of the last confirmed 60-minute candle
  body (`lookahead_off`, `[1]`). OFF by default until validated in replay.
- **Outcome ledger** — day / session / cumulative tallies and reference-only
  P&L (frozen qty × points × point value − cost allowance; ambiguous counted
  as a loss).
- **Prop-eval governor** — reference-equity ledger seeded at the account size,
  daily-loss lockout, EOD/static floor, profit-target progress, budget clamped
  to the daily loss limit.

## What this indicator does not claim

- **No broker fills, no positions, no realized P&L.** "Reference" means a
  signal was recorded; a "reached" level is a candle-range touch, not an
  execution price. All money figures are **reference accounting**.
- **No win-rate, edge or performance claims.** Chop grades are heuristics
  inherited from the original script.
- **No order placement, no broker integration, no strategy conversion.**
- **No repainting**: persistent state commits on confirmed bars only; no
  developing higher-timeframe values are read.
- **No coverage of the loaded-history caveat**: cumulative ledger and governor
  values depend on the chart history TradingView has loaded.

## Input groups (summary)

- **Chop Calculation / Chop History / Dashboard** — lookback, thresholds,
  logging interval, tolerance, history rows, dashboard position/size.
- **CRT Engine** — entry mode (Direct / Re-break / Auto with threshold),
  candles allowed after C2, pullback/stop tick buffers, pause, markers,
  minimum chop grade.
- **CRT Sessions** — enable/exclude sessions (NY 0930-1600 default), Snap vs
  Strict boundary handling, optional day masks (`0930-1600:23456`).
- **CRT Daily Signal Limits** — long/short/combined caps, max concurrent
  references.
- **CRT Estimated Risk** — sizing instrument, custom point value, max risk per
  signal, reserved-risk budget (clamped to the DLL), max contracts, minimum
  gross R:R, cost allowance.
- **CRT Reference Lifecycle** — max observation candles, session-end expiry.
- **CRT 1H Bias Filter** — enable, source (60-minute / chart), mode.
- **Prop-Eval Risk Governor** — account size, profit target, max drawdown,
  drawdown type, daily loss limit, floor mode, stop-at-PT.
- **Alerts** — detailed entry and lifecycle `alert()` toggles.

## Verification

There is no local Pine compiler in this repository. Verification happens in
TradingView: compile the file, then step Playback with the Data Window open.
`crt-nitro-indicator-verification-tests.md` lists every scenario with a status
column; record PASS/FAIL and the replay date per row. Do not treat any version
as stable until the user marks the checklist complete.
