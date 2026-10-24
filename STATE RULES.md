# Stock states - the rules (tl_state v1, tradelab v1.3)

Research defaults declared 2026-10-25 before any real run; settings fingerprint 9b991e86863591c7. Every threshold is transparent and configurable; none is proven. Written by `python tl_state.py rules`.

## How a session is read

1. **Measurements** from the stock's own daily bars up to that session's close (nothing later): swings, EMA20/50/100/200, slopes and distances in ATR units, efficiency, returns, RSI 14, MACD, realised volatility, ATR% rank, gaps, large moves, volume flow.
2. **Swing points** - a swing high is a bar higher than the 3 bars before it and not lower than the 3 after it (swing lows the mirror). It becomes KNOWN 3 sessions after the pivot and is used only from then on; nothing is back-dated to the pivot day.
3. **Raw state** - the first state below (in this order of precedence) whose REQUIRED conditions all hold: S12 > S03 > S01 > S02 > S06 > S08 > S09 > S07. None -> UNKNOWN. Fewer than 60 sessions, or no ATR / EMA50 yet -> INSUFFICIENT_DATA.
4. **Confirmed (primary) state** - changes only after the raw state has been off the current state for 2 sessions in a row; the new state is that session's raw state. The change BEGAN on the first of those sessions and was CONFIRMED on the last (both shown; the confirmation date is when it was known). INSUFFICIENT_DATA ends at once.
5. **Events** (S04, S10, S11, bear trap) and **risk flags** (S05 and others) sit BESIDE the primary state; they never replace it. Volatility is its own dimension.

## Precedence and conflicts

- S12 first: when movement is disorderly the structural reading is unreliable, so nothing else is claimed.
- S03 before S01: a bullish stock moving sideways in a tight range is consolidating, even though its higher-high / higher-low structure still stands.
- S01 before S02: an established structure is a continuation; S02 requires 'not yet established'.
- S02 before S06: once structure and EMA50 have turned, the attempt has become an emerging trend.
- S06 before S08 / S09: a downtrend showing 2+ reversal signs and no fresh low is an attempt; with a fresh low it stays bearish.
- S07 last: no direction and a flat EMA50.
- Nothing fits: UNKNOWN (the card names the two closest states and what each lacks). The engine prefers UNKNOWN to a forced label.
- Missing inputs (EMA200 before 200 sessions, no volume): a condition that cannot be evaluated is neither met nor failed. A REQUIRED condition that cannot be evaluated is not met. Missing supporting / conflicting inputs lower COMPLETENESS, and so confidence.
- Two events at once: the one that changed most recently is shown (a failed breakout replaces its breakout; a later breakdown replaces both); the card lists every event of the last 20 sessions.

## Confidence (not a probability)

- support = share of the state's SUPPORTING conditions that hold (of those that can be evaluated); conflicts = how many CONFLICTING conditions hold; completeness = share of all its conditions that can be evaluated.
- HIGH: support >= 75%, no conflict, completeness >= 90%, in the state 3+ sessions, no change pending.
- LOW: a change pending, or support < 50%, or 2+ conflicts, or completeness < 70%.  MEDIUM: otherwise.  UNKNOWN: state UNKNOWN or INSUFFICIENT_DATA.

## Phase (for the D / ENS research)

- DETERIORATING: a change pending, or support < 50%, or 2+ conflicts.  EARLY: the first 5 sessions of the state.  MATURE: otherwise.

## The primary states

### S01 Strong bullish continuation

- Kind: persistent state.
- Required (all): higher highs and higher lows (the last two confirmed swing highs and lows both rising by 0.25+ ATR); close not more than 0.25 ATR below the last confirmed swing low (bullish structure intact); close above EMA50; EMA50 higher than 10 sessions ago; bullish structure established: held 15+ sessions, or above a rising EMA50 on 60%+ of the 40 sessions before the last 5 (judged on the 15 sessions alone while that context is not yet available).
- Supporting: EMA20 rising (0.3+ ATR over 5 sessions); EMA50 rising (0.5+ ATR over 10 sessions); averages stacked upward (close > EMA20 > EMA50, and > EMA100 > EMA200 where known); efficient rise (20-session efficiency +0.3 or more); 20-session return above 0 and RSI 14 at 50 or more; MACD histogram above 0; above EMA50 on 70%+ of the last 20 sessions; 60-session return above 0; close above EMA200 (needs 200 sessions); volume flow up (signed volume share +0.1 or more over 20 sessions).
- Conflicting (lower confidence): bearish divergence candidate: a higher swing high with RSI 5+ points lower (swing known in the last 20 sessions); price up 5%+ over 20 sessions while volume flow is down (candidate); momentum decelerating (10-session move 1.5+ ATR smaller than the 10 before); close 1.0+ ATR below EMA20; a 2.0+ ATR down candle in the last 5 sessions.
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: a close below the last higher low, or below EMA50.
- Typical transitions: from S02, S03 (after a breakout); to S03, S05 risk, S09 (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

### S02 Emerging bullish trend

- Kind: persistent state (early phase of a trend).
- Required (all): bullish structure forming: higher highs and lows intact, or a higher low then a close above the last swing high; close above EMA50; bullish structure not yet established (see S01).
- Supporting: EMA20 rising (0.3+ ATR over 5 sessions); close above EMA20; efficient rise (20-session efficiency +0.3 or more); 20-session return above 0 and RSI 14 at 50 or more; MACD histogram above 0; last confirmed swing low 0.25+ ATR above the one before; close 0.25+ ATR above the last confirmed swing high (structure broken upward); volume flow up (signed volume share +0.1 or more over 20 sessions); EMA50 higher than 10 sessions ago.
- Conflicting (lower confidence): bearish divergence candidate: a higher swing high with RSI 5+ points lower (swing known in the last 20 sessions); a 2.0+ ATR down candle in the last 5 sessions; EMA50 falling (0.5+ ATR over 10 sessions).
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: a close below the last swing low (the higher low it is built on), or below EMA50.
- Typical transitions: from S06, S07, S08, S09; to S01 (matures), S03, or back to S07 / S09 (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

### S03 Bullish consolidation

- Kind: persistent state.
- Required (all): previously bullish: above a rising EMA50 on 50%+ of the 40 sessions before the last 5; contained range: the last 15 sessions span 3.5 ATR or less with efficiency under 0.3; close not more than 0.25 ATR below the last confirmed swing low (bullish structure intact); close no more than 0.5 ATR below EMA50; not a lower-high / lower-low structure.
- Supporting: last confirmed swing low 0.25+ ATR above the one before; close above EMA50; EMA50 higher than 10 sessions ago; volatility compressed (ATR% in the lowest fifth of its year and short vol below long); volume drying up (10-session median below 0.9x the 50-session); RSI 14 between 45 and 65; above EMA50 on 70%+ of the last 20 sessions.
- Conflicting (lower confidence): a 2.0+ ATR down candle in the last 5 sessions; volume flow down (signed volume share -0.1 or less over 20 sessions); bearish divergence candidate: a higher swing high with RSI 5+ points lower (swing known in the last 20 sessions); EMA20 falling (0.3+ ATR over 5 sessions).
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: a close below the range floor (or the last swing low) ends it downward; a close above the range top resolves it upward.
- Typical transitions: from S01, S02; to S01 with an S04 breakout, or S09 / S10 if the floor breaks (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

### S06 Bullish reversal attempt

- Kind: persistent state (an attempt, not yet a trend).
- Required (all): previously bearish: below a falling EMA50 on 40%+ of the 40 sessions before the last 5; at least 2 reversal signs: higher low, close above the last swing high, bullish divergence, close above a turning EMA20, RSI 50+, volume flow up; no new 25-session low in the last 5 sessions.
- Supporting: last confirmed swing low 0.25+ ATR above the one before; close 0.25+ ATR above the last confirmed swing high (structure broken upward); bullish divergence candidate: a lower swing low with RSI 5+ points higher (swing known in the last 20 sessions); close above EMA20 and EMA20 turning up; RSI 14 at 50 or more; volume flow up (signed volume share +0.1 or more over 20 sessions); price down 5%+ over 20 sessions while volume flow is up (candidate).
- Conflicting (lower confidence): EMA20 falling (0.3+ ATR over 5 sessions); a 2.0+ ATR down candle in the last 5 sessions; volume flow down (signed volume share -0.1 or less over 20 sessions).
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: a new low below the last swing low.
- Typical transitions: from S08, S09; to S02 (confirmed bullish), or back to S08 (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

### S07 Neutral range

- Kind: persistent state.
- Required (all): no direction (20-session efficiency between -0.2 and +0.2); EMA50 flat (less than 0.5 ATR over 10 sessions).
- Supporting: EMA20 flat (less than 0.3 ATR over 5 sessions); neither higher-high/higher-low nor lower-high/lower-low structure; RSI 14 between 40 and 60; 20-session range 8.0 ATR or less; above EMA50 on 30%-70% of the last 20 sessions; volatility not elevated.
- Conflicting (lower confidence): higher highs and higher lows (the last two confirmed swing highs and lows both rising by 0.25+ ATR); lower highs and lower lows (the last two confirmed swing highs and lows both falling by 0.25+ ATR); volatility elevated or extreme; 2+ gaps of 1.0+ ATR in the last 10 sessions.
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: a close outside the 20-session range.
- Typical transitions: from any trend losing direction; to S02 / S09 on a break of the range, S03 (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

### S08 Bearish continuation

- Kind: persistent state.
- Required (all): lower highs and lower lows (the last two confirmed swing highs and lows both falling by 0.25+ ATR); close not more than 0.25 ATR above the last confirmed swing high (bearish structure intact); close below EMA50; EMA50 lower than 10 sessions ago; bearish structure established: held 15+ sessions, or below a falling EMA50 on 60%+ of the 40 sessions before the last 5 (judged on the 15 sessions alone while that context is not yet available).
- Supporting: EMA20 falling (0.3+ ATR over 5 sessions); EMA50 falling (0.5+ ATR over 10 sessions); averages stacked downward (close < EMA20 < EMA50, and < EMA100 < EMA200 where known); efficient fall (20-session efficiency -0.3 or less); 20-session return below 0 and RSI 14 below 50; MACD histogram below 0; above EMA50 on 30% or less of the last 20 sessions; 60-session return below 0; close below EMA200 (needs 200 sessions); volume flow down (signed volume share -0.1 or less over 20 sessions).
- Conflicting (lower confidence): bullish divergence candidate: a lower swing low with RSI 5+ points higher (swing known in the last 20 sessions); price down 5%+ over 20 sessions while volume flow is up (candidate); momentum accelerating (10-session move 1.5+ ATR larger than the 10 before); close 1.0+ ATR above EMA20.
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: a close above the last lower high, or above EMA50.
- Typical transitions: from S09; to S06, S07 (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

### S09 Emerging bearish trend

- Kind: persistent state (early phase of a decline).
- Required (all): bearish structure forming: lower highs and lows intact, or a close below the last swing low with a lower high or below EMA50; bearish structure not yet established (see S08); close below EMA50 or EMA20 falling.
- Supporting: EMA20 falling (0.3+ ATR over 5 sessions); close below EMA20; efficient fall (20-session efficiency -0.3 or less); 20-session return below 0 and RSI 14 below 50; MACD histogram below 0; last confirmed swing high 0.25+ ATR below the one before; close 0.25+ ATR below the last confirmed swing low (structure broken downward); volume flow down (signed volume share -0.1 or less over 20 sessions); a 2.0+ ATR down candle in the last 5 sessions; EMA50 lower than 10 sessions ago.
- Conflicting (lower confidence): bullish divergence candidate: a lower swing low with RSI 5+ points higher (swing known in the last 20 sessions); momentum accelerating (10-session move 1.5+ ATR larger than the 10 before); EMA50 rising (0.5+ ATR over 10 sessions).
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: a close back above the last swing high, or above EMA50.
- Typical transitions: from S01, S02, S03, S07; to S08 (matures), S06, or back to S07 / S02 (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

### S12 Unstable / disorderly

- Kind: persistent state (usually short-lived).
- Required (all): disorderly: 10-session vol 1.8x+ the 60-session or ATR% at its 95% rank, AND 3+ reversals of large moves in 20 sessions or 4+ large gaps in 10, AND efficiency under 0.25.
- Supporting: volatility extreme; 2+ gaps of 1.0+ ATR in the last 10 sessions; a 2.0+ ATR down candle in the last 5 sessions; no direction (20-session efficiency between -0.2 and +0.2).
- Conflicting (lower confidence): higher highs and higher lows (the last two confirmed swing highs and lows both rising by 0.25+ ATR); lower highs and lower lows (the last two confirmed swing highs and lows both falling by 0.25+ ATR).
- Confirmation: 2 sessions in a row (as every primary state).
- Invalidation: 10-session volatility back under 1.8x the 60-session and fewer large reversals.
- Typical transitions: from any state; to S07, UNKNOWN or a new trend once volatility calms (possible paths, not mandatory).
- Minimum history: 60 sessions to be read at all; the 'previously bullish / bearish' context (needed by S03 and S06) from about 85 sessions; EMA100 / EMA200 conditions from 100 / 200 sessions.

## Events and risk flags

- **S04**: BREAKOUT - the close crosses above the highest high of the previous 55 sessions (else 20). NEW on that session; CONFIRMED when every close of the next 3 sessions stays at or above that level; STALLED when neither confirmed nor failed within 5 sessions. Temporary event (shown for 10 sessions).
- **S11**: FAILED BREAKOUT / BULL TRAP - within 5 sessions of a breakout (even a confirmed one) a close falls more than 0.25 ATR below the breakout level. Temporary event (shown for 10 sessions).
- **S10**: BREAKDOWN - the close crosses below the lowest low of the previous 55 sessions (else 20); NEW / CONFIRMED (3 sessions held below) / STALLED as for S04. Temporary event.
- **bear trap**: FAILED BREAKDOWN - within 5 sessions of a breakdown a close rises more than 0.25 ATR above the level.
- **S05**: OVEREXTENDED (risk flag, not a sell signal) - close 3.0+ ATR above EMA20, or 6.0+ ATR above EMA50, or a 6.0+ ATR rise in 10 sessions; EXTREME at 4.5 / 9.0 ATR. Shown beside any primary state.
- Other risk flags: STRETCHED_DOWN = stretched far below EMA20; BIG_DOWN_CANDLE = large down candle in the last 5 sessions; GAP_RISK = repeated large gaps; BEAR_DIVERGENCE = bearish divergence candidate (RSI); PRICE_UP_FLOW_DOWN = price up, volume flow down (candidate).
- Notes: BULL_DIVERGENCE = bullish divergence candidate (RSI); PRICE_DOWN_FLOW_UP = price down, volume flow up (candidate); VOL_COMPRESSED = volatility compressed.
- Volatility: EXTREME if 10-session vol >= 2.2x the 60-session or ATR% at its 97% rank of 250 sessions; ELEVATED at 1.5x / 85%; COMPRESSED at rank <= 20% with ratio <= 0.8; else NORMAL.
- Volume-based signs (flow, divergence with price) are CANDIDATES read from OHLCV; they never establish accumulation or distribution.

## Every condition

- `above20`: close above EMA20
- `above200`: close above EMA200 (needs 200 sessions)
- `above20_turning`: close above EMA20 and EMA20 turning up
- `above50`: close above EMA50
- `above_last_low`: close not more than 0.25 ATR below the last confirmed swing low (bullish structure intact)
- `accel`: momentum accelerating (10-session move 1.5+ ATR larger than the 10 before)
- `align_bear`: averages stacked downward (close < EMA20 < EMA50, and < EMA100 < EMA200 where known)
- `align_bull`: averages stacked upward (close > EMA20 > EMA50, and > EMA100 > EMA200 where known)
- `bear_div`: bearish divergence candidate: a higher swing high with RSI 5+ points lower (swing known in the last 20 sessions)
- `bear_early`: bearish structure forming: lower highs and lows intact, or a close below the last swing low with a lower high or below EMA50
- `bear_est`: bearish structure established: held 15+ sessions, or below a falling EMA50 on 60%+ of the 40 sessions before the last 5 (judged on the 15 sessions alone while that context is not yet available)
- `bear_not_est`: bearish structure not yet established (see S08)
- `bear_struct`: lower highs and lower lows (the last two confirmed swing highs and lows both falling by 0.25+ ATR)
- `below20`: close below EMA20
- `below200`: close below EMA200 (needs 200 sessions)
- `below50`: close below EMA50
- `below_last_high`: close not more than 0.25 ATR above the last confirmed swing high (bearish structure intact)
- `big_down`: a 2.0+ ATR down candle in the last 5 sessions
- `bos_dn`: close 0.25+ ATR below the last confirmed swing low (structure broken downward)
- `bos_up`: close 0.25+ ATR above the last confirmed swing high (structure broken upward)
- `bull_div`: bullish divergence candidate: a lower swing low with RSI 5+ points higher (swing known in the last 20 sessions)
- `bull_early`: bullish structure forming: higher highs and lows intact, or a higher low then a close above the last swing high
- `bull_est`: bullish structure established: held 15+ sessions, or above a rising EMA50 on 60%+ of the 40 sessions before the last 5 (judged on the 15 sessions alone while that context is not yet available)
- `bull_not_est`: bullish structure not yet established (see S01)
- `bull_struct`: higher highs and higher lows (the last two confirmed swing highs and lows both rising by 0.25+ ATR)
- `contained`: contained range: the last 15 sessions span 3.5 ATR or less with efficiency under 0.3
- `decel`: momentum decelerating (10-session move 1.5+ ATR smaller than the 10 before)
- `deep_below20`: close 1.0+ ATR below EMA20
- `disorder`: disorderly: 10-session vol 1.8x+ the 60-session or ATR% at its 95% rank, AND 3+ reversals of large moves in 20 sessions or 4+ large gaps in 10, AND efficiency under 0.25
- `eff_dn`: efficient fall (20-session efficiency -0.3 or less)
- `eff_flat`: no direction (20-session efficiency between -0.2 and +0.2)
- `eff_up`: efficient rise (20-session efficiency +0.3 or more)
- `ema50_dn`: EMA50 lower than 10 sessions ago
- `ema50_up`: EMA50 higher than 10 sessions ago
- `fall20`: EMA20 falling (0.3+ ATR over 5 sessions)
- `fall50`: EMA50 falling (0.5+ ATR over 10 sessions)
- `flat20`: EMA20 flat (less than 0.3 ATR over 5 sessions)
- `flat50`: EMA50 flat (less than 0.5 ATR over 10 sessions)
- `flow_dn`: volume flow down (signed volume share -0.1 or less over 20 sessions)
- `flow_up`: volume flow up (signed volume share +0.1 or more over 20 sessions)
- `fresh_low`: made the lowest low of 25 sessions within the last 5
- `gap_risk`: 2+ gaps of 1.0+ ATR in the last 10 sessions
- `hh`: last confirmed swing high 0.25+ ATR above the one before
- `hl`: last confirmed swing low 0.25+ ATR above the one before
- `lh`: last confirmed swing high 0.25+ ATR below the one before
- `ll`: last confirmed swing low 0.25+ ATR below the one before
- `macd_dn`: MACD histogram below 0
- `macd_up`: MACD histogram above 0
- `mom_dn`: 20-session return below 0 and RSI 14 below 50
- `mom_up`: 20-session return above 0 and RSI 14 at 50 or more
- `near50`: close no more than 0.5 ATR below EMA50
- `no_bear_struct`: not a lower-high / lower-low structure
- `no_fresh_low`: no new 25-session low in the last 5 sessions
- `no_struct`: neither higher-high/higher-low nor lower-high/lower-low structure
- `persist_dn`: above EMA50 on 30% or less of the last 20 sessions
- `persist_mid`: above EMA50 on 30%-70% of the last 20 sessions
- `persist_up`: above EMA50 on 70%+ of the last 20 sessions
- `prior_bear`: previously bearish: below a falling EMA50 on 40%+ of the 40 sessions before the last 5
- `prior_bull`: previously bullish: above a rising EMA50 on 50%+ of the 40 sessions before the last 5
- `pv_dn_flow_up`: price down 5%+ over 20 sessions while volume flow is up (candidate)
- `pv_up_flow_dn`: price up 5%+ over 20 sessions while volume flow is down (candidate)
- `r60_dn`: 60-session return below 0
- `r60_up`: 60-session return above 0
- `range20_ok`: 20-session range 8.0 ATR or less
- `rev2`: at least 2 reversal signs: higher low, close above the last swing high, bullish divergence, close above a turning EMA20, RSI 50+, volume flow up
- `rise20`: EMA20 rising (0.3+ ATR over 5 sessions)
- `rise50`: EMA50 rising (0.5+ ATR over 10 sessions)
- `rsi_cons`: RSI 14 between 45 and 65
- `rsi_mid`: RSI 14 between 40 and 60
- `rsi_up50`: RSI 14 at 50 or more
- `vol_comp`: volatility compressed (ATR% in the lowest fifth of its year and short vol below long)
- `vol_drying`: volume drying up (10-session median below 0.9x the 50-session)
- `vol_elev`: volatility elevated or extreme
- `vol_extreme`: volatility extreme
- `vol_not_elev`: volatility not elevated
- `weak_trend`: close below EMA50 or EMA20 falling
- `well_above20`: close 1.0+ ATR above EMA20

## Settings (STATE_CFG)

```
{
 "declared": "2026-10-25",
 "min_bars": 60,
 "atr_n": 14,
 "ema": [
  20,
  50,
  100,
  200
 ],
 "slope20_lag": 5,
 "slope20_min": 0.3,
 "slope50_lag": 10,
 "slope50_min": 0.5,
 "swing_k": 3,
 "swing_tol_atr": 0.25,
 "eff_n": 20,
 "eff_trend": 0.3,
 "eff_flat": 0.2,
 "persist_n": 20,
 "persist_hi": 0.7,
 "persist_lo": 0.3,
 "established": 15,
 "ctx_n": 40,
 "ctx_lag": 5,
 "ctx_est": 0.6,
 "ctx_cons": 0.5,
 "ctx_prior": 0.4,
 "cons_n": 15,
 "cons_range_atr": 3.5,
 "cons_eff": 0.3,
 "cons_ema50_atr": 0.5,
 "rev_min": 2,
 "fresh_low_n": 5,
 "fresh_low_window": 25,
 "div_rsi_pts": 5.0,
 "div_recent": 20,
 "breakout_windows": [
  55,
  20
 ],
 "hold_n": 3,
 "fail_n": 5,
 "fail_atr": 0.25,
 "event_show": 10,
 "ext20_atr": 3.0,
 "ext50_atr": 6.0,
 "ext10_atr": 6.0,
 "extreme20_atr": 4.5,
 "extreme50_atr": 9.0,
 "stretch_dn_atr": -3.0,
 "rv_short": 10,
 "rv_long": 60,
 "rank_n": 250,
 "rank_min": 60,
 "vol_elev_ratio": 1.5,
 "vol_elev_rank": 0.85,
 "vol_ext_ratio": 2.2,
 "vol_ext_rank": 0.97,
 "vol_comp_rank": 0.2,
 "vol_comp_ratio": 0.8,
 "big_move_atr": 1.5,
 "flip_n": 20,
 "gap_atr": 1.0,
 "gap_n": 10,
 "gap_risk": 2,
 "down_atr": 2.0,
 "down_n": 5,
 "disorder_ratio": 1.8,
 "disorder_rank": 0.95,
 "disorder_flips": 3,
 "disorder_gaps": 4,
 "disorder_eff": 0.25,
 "range20_atr": 8.0,
 "rvol_n": 50,
 "rvol_min": 20,
 "flow_n": 20,
 "flow_min": 0.1,
 "pv_ret": 0.05,
 "dry_ratio": 0.9,
 "decel_atr": -1.5,
 "accel_atr": 1.5,
 "below20_atr": -1.0,
 "above20_atr": 1.0,
 "confirm_n": 2,
 "early_n": 5,
 "weak_support": 0.5,
 "conflicts_low": 2,
 "high_support": 0.75,
 "high_age": 3,
 "complete_high": 0.9,
 "complete_low": 0.7,
 "history_keep": 6
}
```
