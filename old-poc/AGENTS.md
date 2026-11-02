# AGENTS.md — CRT Nitro indicator: working state

Living state file for any agent (or human) resuming work on this repo.
`crt-nitro-indicator-build-plan.md` is the original handoff plan; this file
supersedes its "current state" sections and records what was actually done.
Update this file at every phase boundary.

Last updated: 2026-10-13 — all code phases (1–5) complete and committed;
Phase 6 docs complete. Next: TradingView compile + Playback verification by the
user (see checklist). No further code changes should be made until that
verification pass reports back, so the verified artifact stays identifiable.

## Repo map

| File | Role | State |
| --- | --- | --- |
| `crt-nitro-indicator.pine` | The indicator (single source of truth, `//@version=6`) | v0.9.0-alpha.1 applied in place |
| `AGENTS.md` | This file: living status | maintained per phase |
| `crt-nitro-indicator-build-plan.md` | Original handoff/completion plan (phases, defect register D-01…D-10, decisions Q1–Q7) | reference; do not edit |
| `crt-nitro-indicator-reqs.md` | User requirements | reference |
| `new-crt-nitro-indicator-session.md` | Session transcript / release notes 0.1.0→0.5.0 | reference |
| `crt-nitro-indicator-verification-tests.md` | Verification checklists | will be rewritten as a living checklist in Phase 6 |
| `old-crt-indicator-issues.md` | Defect list of the original indicator (bias/HTF issues) | reference |

## Hard rules (do not break)

1. Pine Script v6. Keep `//@version=6` as the first line.
2. Never use `text`, `label`, `line`, `table`, `color`, `size`, `style` as identifiers.
   `f_cell` takes `cellText`; `f_crtUpdateLabel` takes `labelText`.
3. Persistent state mutates only under `barstate.isconfirmed`. Dashboard/drawings
   render on `barstate.islast` from committed state.
4. No developing HTF value reads: `request.security(..., lookahead = barmerge.lookahead_off)`
   with a `[1]` confirmed-bar offset.
5. Indicator, not strategy. No broker integration, orders, or fill claims.
6. No performance/win-rate/edge claims anywhere.
7. Version must be updated in the same edit in: header comment, `indicator()` title,
   dashboard cells, and every alert string.
8. Edit `crt-nitro-indicator.pine` in place; no chat-window chunking.
9. Supporting docs (`README.md`, `CHANGELOG.md`, verification checklist) update with
   the phase that changes them.

## Decisions (confirmed by the user 2026-10-13, all plan defaults)

| # | Decision | Answer |
| --- | --- | --- |
| Q1 | 5m charts | **Chop diagnostics only**; CRT engine requires 15m/30m/60m. Guard text keeps "5m, 15m, 30m or 60m" as supported charts. |
| Q2 | Concurrency | **2 max, one per direction.** Same-direction re-entry only after the prior same-direction reference ends. |
| Q3 | Risk limit | **DLL = $300** is the hard lockout; reserved-risk budget default drops to $300 and is clamped to never exceed the DLL. |
| Q4 | 1H bias | Bias = sign of the last **confirmed** 1H candle body (`close - open`), `lookahead_off` + `[1]`; require sweep alignment; **OFF by default** until validated in replay. |
| Q5 | Ledger | **Yes, reference-only P&L**: frozen qty × points × point value − `crtCostAllowance` × qty per resolved side-pair; per-day and cumulative; labelled "reference, not fills". |
| Q6 | Drawdown | **EOD** peak-of-reference-equity at trading-day close. |
| Q7 | Sizing default | **Micro equivalent** (ES chart → MES risk, etc.). |

## Phase status

| Phase | Version | Status | Notes |
| --- | --- | --- | --- |
| 1 | `0.5.1-alpha.1` | **Code complete — TradingView compile check pending** | D-01…D-10 |
| 2 | `0.6.0-alpha.1` | **Code complete — TradingView compile check pending** | per-direction reference slots, per-direction locks, 5m engine gate |
| 3 | `0.7.0-alpha.1` | **Code complete — TradingView compile check pending** | 1H bias filter (OFF by default, `lookahead_off` + `[1]`, `BIAS_BLOCKED`) |
| 4 | `0.8.0-alpha.1` | **Code complete — TradingView compile check pending** | outcome ledger (day/session/cumulative), lifecycle alerts, tally plots |
| 5 | `0.9.0-alpha.1` | **Code complete — TradingView compile check pending** | prop-eval governor: DLL lockout, EOD/static floor, PT progress, budget clamp |
| 6 | docs | **done** | living verification checklist rewritten (PENDING rows), CHANGELOG.md, README.md |

`1.0.0` is user-gated: promote only when the user marks the verification
checklist complete.

## Defect register status (from the build plan)

| ID | Description | Status |
| --- | --- | --- |
| D-01 | Terminal candle could not arm a new candidate | Fixed in 0.5.1: `mayArmThisBar` falls through after non-TRIGGERED terminal states |
| D-02 | Quotas unreachable / 60m near-dead | Part 1 (Snap/Strict boundary mode) + Part 3 (NY default 0930-1600) in 0.5.1; Part 2 (per-direction concurrency, slots, per-slot drawings) in 0.6.0 |
| D-03 | Auto mode silent dead end | Fixed in 0.5.1: `crtMaxBars` default 3, Auto threshold input, Auto falls back to Direct with a note |
| D-04 | Rejected candidates repainted dashboard | Fixed in 0.5.1: candidate levels commit only on `WAIT_BREAK` |
| D-05 | Duplicated chop thresholds | Fixed in 0.5.1: `CHOP_A..CHOP_D` constants + "Minimum chop grade for entry" input |
| D-06 | Positional session parsing, day mask ignored | Fixed in 0.5.1: validated parser, day masks honoured, "Invalid session string" surfaced |
| D-07 | Score denominators off by one | Fixed in 0.5.1: `pairSamples = lookback - 1` |
| D-08 | Dead state / chunk markers | Fixed in 0.5.1: `crtLastPlanEndBar` deleted, guard branch commented, PART markers removed |
| D-09 | Trading-day label timezone mismatch | Fixed in 0.5.1: rendered in `syminfo.timezone` ("Trading day (exchange timezone)") |
| D-10 | Duplicate issues file | Done: `old-crt-nitro-indicator-issues.md` deleted |

## Verification state

- There is **no local Pine compiler**. Every compile/behavior claim must come from the
  user pasting the `.pine` into TradingView and stepping Playback.
- User previously compiled 0.3.1, 0.4.0, 0.5.0 successfully; behavioral verification
  of 0.5.0 was partial when this work resumed.
- **Nothing from 0.5.1 or later is compile- or behavior-verified yet.**
- The living checklist is `crt-nitro-indicator-verification-tests.md`; it now covers
  compilation guards, chop, sessions, sequencing, lifecycle, counters, concurrency,
  bias, ledger, governor and regression rows, each with a PENDING status.
- Git history carries one commit per phase (`0.5.1`, `0.6.0`, `0.7.0`, `0.8.0`,
  `0.9.0`, docs) so any phase can be compiled independently in TradingView.

## Next actions

1. User: paste `crt-nitro-indicator.pine` (v0.9.0-alpha.1) into TradingView, compile
   (checklist row C-01).
2. User: run the replay recipes below and mark checklist rows PASS/FAIL with dates.
3. Agent: on FAIL reports, fix on top of the failed version and re-commit; on PASS
   reports, promote to `0.9.1-beta.1` only when the user requests it.
4. `1.0.0` (and dropping the UNSTABLE cell) stays user-gated.

## Replay recipes (prepare per phase; minimum set)

| Phase | Scenario |
| --- | --- |
| 1 | 15m NQ replay: a failing setup followed immediately by a one-sided sweep must arm on that same candle (`WAIT_BREAK`). |
| 1 | 60m chart, NY session: "Eligible candles / session" cell must reach ≥ 6 before the session ends; boundary mode shows `Snap`. |
| 1 | Auto mode, defaults: no instant `REJECTED`; grade-B sweep shows `Re-break` (window ≥ 3) and grade-A shows `Direct`. |
| 1 | Set a session string to garbage (e.g. `abc`): dashboard shows "Invalid session string" and no signals. |
| 2 | A LONG reference active while a SHORT sweep forms: both must arm; dashboard shows one reference row per direction. |
| 2 | Fourth signal of the day under the 3-combined cap must be `DAILY_CAP`. |
| 3 | Long candidate under a confirmed-down 1H candle must be `BIAS_BLOCKED` (filter enabled). |
| 4 | A candle whose range contains both stop and target must be `AMBIGUOUS`, counted as a loss, one alert. |
| 5 | A day whose reference P&L breaches the DLL must show `DLL_LOCKOUT` and arm nothing further that day. |

## Known deviations / design notes recorded during implementation

- `Maximum concurrent references` input will be **1–2** in Phase 2 (plan said 1–4):
  under Q2 ("one per direction") effective concurrency cannot exceed 2, so a 3–4
  option would be misleading. Revisit only if the user requests stacking.
- Per-slot storage is parallel arrays indexed 0 = LONG / 1 = SHORT. A slot keeps its
  ended state (TARGET_REACHED/STOP_REACHED/AMBIGUOUS/EXPIRED/SESSION_ENDED/DAY_ENDED)
  until the next admission in that direction or the daily clear. `refMode` was dropped
  to avoid write-only state; the candidate's mode is still shown on the dashboard.
- With `crtMaxConcurrent = 1` the engine gates on **any** active/just-ended reference;
  admission and cancellation evaluate the same rule so the two cannot disagree.
- 5m charts: chop, session, history and dashboard render; `engineTF` gates the
  lifecycle and entry engine off (Q1).
- The wider NY default (0930-1600 RTH) ships without a separate "NY AM" sub-window
  input (plan allowed either approach).
- Out-of-window DST shifts for overnight sessions: the pre-start shift uses a
  fixed 24h millisecond offset; a DST transition inside the window can shift the
  boundary by one hour. Acceptable; documented in code.
- Lifecycle observations are references, not fills: "reached" means the candle
  range touched the level, not an execution price (existing design, kept).
- Ledger semantics (Phase 4): resolved outcomes (target / stop / ambiguous) carry
  reference P&L; ambiguous is priced at the stop (counted as a loss) and shown
  separately; unresolved endings (expired / session end / day end) count as
  outcomes with zero P&L and no cost. Costs apply only to resolved outcomes.
  Cumulative values depend on loaded history; every ledger cell says "reference".
- Governor semantics (Phase 5): reference equity = account size + cumulative net
  reference P&L (history-dependent). DLL lockout blocks/cancels candidates only;
  active references keep being monitored, and the lockout persists across session
  boundaries until the trading-day reset. Floor = max(peak EOD ref equity - DD,
  size - DD) for the trailing mode, size - DD for static; intraday mode updates the
  peak continuously on confirmed bars. Q3 applied: budget default $300 and clamped
  to the DLL.

## Constraints carried from the build plan (still binding)

- ES/MES, NQ/MNQ, YM/MYM, RTY/M2K micro point values (MNQ 2.0, MES 5.0, MYM 0.5,
  M2K 5.0) were spot-checked and are correct — do not "correct" them.
- Non-defects: entry requires a **close** beyond C2's extreme; a candle sweeping both
  sides of C1 is rejected; reserved risk is never refunded; `TARGET_CONSUMED` rejects
  a setup whose target was tagged before entry; both-levels-reached is `AMBIGUOUS`,
  never a win.
- Post-1.0 backlog lives in section 7 of the build plan; do not start it early.
