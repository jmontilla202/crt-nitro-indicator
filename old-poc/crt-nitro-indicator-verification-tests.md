# CRT Nitro Indicator — Living Verification Checklist

Version under test: **0.9.0-alpha.1** (`crt-nitro-indicator.pine`).
This file supersedes the per-version append-only checklists (0.1.0 → 0.5.0).
Earlier notes claimed "compiles / partially verified"; those claims are folded in
below as rows and must be re-marked against this file.

Status values: **PENDING** / **PASS** / **FAIL** / **N/A**. Record the Playback
date and a one-line note when a row is marked. Do not mark the script stable
until every non-N/A row is PASS or an accepted N/A.

## How to verify

1. Paste `crt-nitro-indicator.pine` into the TradingView Pine Editor, Save,
   and add it to a chart (indicator, not strategy).
2. Use Bar Replay (Playback) on the instrument/timeframe named in the row.
3. The dashboard cells and the **Data Window** series carry every committed
   state variable; step candles and read them there. No local Pine compiler
   exists in this repo — TradingView is the only verifier.
4. Recreate TradingView alerts after any settings change.
5. Solicit no behavior claims from the code alone; mark rows from observed replay.

## 0. Compilation and chart guards

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| C-01 | Compile 0.9.0-alpha.1 in the Pine Editor | Saves with no errors; dashboard renders | PENDING | | |
| C-02 | Switch to a 1m chart | "Use a 5m, 15m, 30m or 60m timeframe" replaces the dashboard body | PENDING | | |
| C-03 | Switch to a 5m chart | Dashboard renders; permission cell reads "5m: chop diagnostics only; engine needs 15m+"; no candidates ever appear | PENDING | | |
| C-04 | Non-standard chart (Heikin Ashi), 15m | "Use standard time-based candles" replaces the dashboard body | PENDING | | |

## 1. Chop module (0.1.x scope)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| CH-01 | Watch the open candle | Latest closed score does not change intrabar | PENDING | | |
| CH-02 | Move to a lower logged score | Sampled trend reads Improving, subject to the tolerance | PENDING | | |
| CH-03 | Wait for the logging interval | History rows append at the configured interval | PENDING | | |
| CH-04 | Set "Minimum chop grade for entry" = A | A/B pass text shows "A chop pass"; a B-grade setup is Chop blocked | PENDING | | |
| CH-05 | Grade color/letter boundaries | A < 15, B < 30, C < 45, D < 60, F ≥ 60 exactly as constants | PENDING | | |

## 2. Sessions

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| S-01 | 60m chart, NY default 0930-1600, Snap | "Eligible candles / session" reaches ≥ 6 by session end | PENDING | | |
| S-02 | Same chart, Strict mode | Candles not fully inside 0930-1600 are excluded (counter lower) | PENDING | | |
| S-03 | Replace NY hours with `abc` | Permission reads "Invalid session string"; no candidates | PENDING | | |
| S-04 | Append day mask `:23456` to NY hours | Session applies Mon–Fri only (verify on a Saturday replay) | PENDING | | |
| S-05 | Cross a session boundary | Eligible-candle counter resets; a new session can arm again | PENDING | | |

## 3. Entry sequencing (0.2.x scope)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| E-01 | C2 sweeps one side, closes inside C1 | WAIT_BREAK with chop A/B and valid geometry | PENDING | | |
| E-02 | Later close beyond the C2 break level (Direct) | TRIGGERED, marker, entry alert (age within `crtMaxBars`, default 3) | PENDING | | |
| E-03 | No break close before the window elapses | EXPIRED, note "Entry window elapsed" | PENDING | | |
| E-04 | Candle sweeps both sides of C1 | No setup armed | PENDING | | |
| E-05 | Chop drops below the minimum grade while waiting | CANCELLED; note names the grade (e.g. "below B") | PENDING | | |
| E-06 | C1 target touched before entry confirmation | TARGET_CONSUMED, no admission | PENDING | | |
| E-07 | Re-break mode, `crtMaxBars ≥ 3` | Break → pullback close → re-break close on three separate candles | PENDING | | |
| E-08 | Re-break forced with `crtMaxBars < 3` | REJECTED with the 3-candle note (explicit mode only) | PENDING | | |
| E-09 | Auto mode, defaults | Grade B (score ≥ threshold) → Re-break; grade A → Direct; no instant REJECTED | PENDING | | |
| E-10 | Auto mode with threshold above the score | Direct is selected; no fallback note | PENDING | | |
| E-11 | Failed setup followed immediately by a valid one-sided sweep | New candidate arms on that same candle (same-candle re-arm) | PENDING | | |

## 4. Lifecycle (0.5.x scope)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| L-01 | Entry candle itself | Reference stays ACTIVE; no outcome evaluated on its entry candle | PENDING | | |
| L-02 | Later candle tags only the target / only the stop | TARGET_REACHED / STOP_REACHED on the confirming close only | PENDING | | |
| L-03 | One candle's range contains both levels | AMBIGUOUS ("Both reached; order unknown"), counted as a loss | PENDING | | |
| L-04 | Temporarily set observation limit to a small value | Level event takes precedence; otherwise EXPIRED at the limit | PENDING | | |
| L-05 | Session-end expiry ON, leave the entry session | SESSION_ENDED on the first candle not fully inside the entry session | PENDING | | |
| L-06 | Cross the trading-day boundary | DAY_ENDED; with "Clear reference plan on new trading day" OFF the status is retained | PENDING | | |
| L-07 | End any reference | Reserved signal risk is not refunded | PENDING | | |
| L-08 | Reference ends on a candle that is also a valid sweep | Same-direction candidate does not arm on that candle | PENDING | | |

## 5. Counters, quotas, drawings (0.3.x/0.4.x scope)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| Q-01 | First qualified long | LONG count 1; SHORT unchanged; drawings frozen | PENDING | | |
| Q-02 | Fourth signal of the day (default caps 2/2/3) | DAILY_CAP; no marker, plan or alert | PENDING | | |
| Q-03 | Qualified short after a long cap | Permitted while its own quota and the combined cap remain | PENDING | | |
| Q-04 | Rejected/cancelled setups | No counter increment, no reserved-risk change | PENDING | | |
| Q-05 | Risk sizing (NQ → MNQ $2/pt) | 20-pt stop + $3 allowance = $43/contract; $150 cap admits 3 contracts | PENDING | | |
| Q-06 | RR_BLOCKED case | No slot consumed, no risk reserved, no drawings changed | PENDING | | |
| Q-07 | Rejected candidate dashboard block | Candidate rows keep the last admitted values (no repaint) | PENDING | | |
| Q-08 | Trading-day reset | Directional counts and reserved risk reset together | PENDING | | |

## 6. Directional concurrency (0.6.0)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| CC-01 | LONG reference active while a valid SHORT sweep forms | SHORT candidate arms and can trigger; both references live | PENDING | | |
| CC-02 | Same-direction candidate while that reference is active | Blocked; permission cell reads "LONG reference active; SHORT available" | PENDING | | |
| CC-03 | Set maximum concurrent references = 1 | Any active reference blocks all new candidates | PENDING | | |
| CC-04 | Both references with plans on chart | Six lines/labels present; ended references stop extending | PENDING | | |
| CC-05 | Both references end | Each ends independently; tallies record each outcome once | PENDING | | |

## 7. 1H bias filter (0.7.0)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| B-01 | Filter disabled (default) | No BIAS_BLOCKED states; dashboard reads "Disabled / Inactive" | PENDING | | |
| B-02 | Enable, Require alignment, confirmed-down 1H | LONG candidate → BIAS_BLOCKED; no counter increment | PENDING | | |
| B-03 | Flat 1H bias | Require alignment blocks both directions; Block counter-bias allows both | PENDING | | |
| B-04 | Bias never changes intrabar | Bias cell constant during the forming candle; matches the prior candle on a 60m chart | PENDING | | |
| B-05 | Bias turns counter during a pending sequence | Pending candidate shows CANCELLED ("1H bias now opposes this setup") | PENDING | | |

## 8. Outcome ledger and lifecycle alerts (0.8.0)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| LE-01 | Target outcome | Day/session/cumulative target buckets +1; one lifecycle alert | PENDING | | |
| LE-02 | Stop outcome | Stop bucket +1; net P&L negative; one alert | PENDING | | |
| LE-03 | Ambiguous candle | Ambiguous bucket +1, priced as a loss; stop/ambiguous alert fires | PENDING | | |
| LE-04 | Expired / session / day endings | Unresolved buckets +1, zero P&L, no cost | PENDING | | |
| LE-05 | Resets | Day buckets reset at the trading-day boundary; session buckets on session change; cumulative persists | PENDING | | |
| LE-06 | Data Window step-through | Series values reproduce the dashboard cells while stepping candles | PENDING | | |
| LE-07 | Alert frequency | Exactly one alert per terminal event, on the confirming close | PENDING | | |

## 9. Prop-eval risk governor (0.9.0)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| G-01 | Defaults 50k / 3k / 2k / EOD / 300 | Risk-budget cell shows $300 (clamped); governor reads "Active" | PENDING | | |
| G-02 | Replay day whose reference P&L reaches −$300 | DLL_LOCKOUT; nothing further arms that day; lockout survives a session change | PENDING | | |
| G-03 | Floor interaction | A signal whose one-contract risk exceeds floor headroom → GOV_FLOOR_BLOCKED even with quota left | PENDING | | |
| G-04 | Profit target | "% to profit target" cell; enable stop-at-PT → PT_REACHED blocks new sequences | PENDING | | |
| G-05 | Intraday vs EOD peak | Intraday mode updates the peak continuously; EOD mode only at the day boundary | PENDING | | |
| G-06 | Reference disclaimers | Every governor cell/label carries "reference"; no broker P&L wording | PENDING | | |

## 10. Regression against earlier failures (original indicator's issues)

| ID | Test | Expected | Status | Date | Notes |
| --- | --- | --- | --- | --- | --- |
| R-01 | Consecutive valid C2 candles | Both are seen (no false-to-true transition loss) | PENDING | | |
| R-02 | Developing HTF values | Never read: bias uses confirmed `[1]` values only | PENDING | | |
| R-03 | One candle, full break/pullback/re-break | Sequence stages cannot all pass on one candle | PENDING | | |
| R-04 | TP-before-SL defect | A candle touching both is AMBIGUOUS, never a win | PENDING | | |
| R-05 | Chop verdict vs engine gate | Displayed chop verdict and admission agree | PENDING | | |
| R-06 | Stop placement | Structural (C2 opposite extreme ± buffer); invalid geometry rejected | PENDING | | |

## Promotion gate

`0.9.1-beta.1` / `1.0.0` are user-gated: promote only when every row above is
PASS (or an accepted N/A) and the user explicitly requests it. The `UNSTABLE`
dashboard cell is dropped only at that point.
