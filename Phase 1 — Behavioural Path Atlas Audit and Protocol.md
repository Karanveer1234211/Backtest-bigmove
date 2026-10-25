# Phase 1 — Behavioural Path Atlas: Audit and Protocol

Oct 25, 2026 · @karan

## Verdict first

Phase 1 was revised on 25 Oct (afternoon) with nine methodological changes from your review, and the revised protocol is **not frozen yet**. Nothing below is a finding: no path outcome has been looked at. pathlab is written and tested on synthetic worlds; only its outcome-blind coverage audit is released, and everything else runs after you freeze.

The revised sequence:

1. **Audit coverage first.** For every state × regime, year by year and in every candidate training and test window: stock-days, stocks, market dates, separate 10-session stretches, and an effective sample computed from a value known at T0. No outcome is read. Fold boundaries and minimum supports are fixed from these counts, then frozen.
2. **Discover the paths.** The daily and 3-minute five-session paths of every eligible equity (ETFs counted apart), grouped on two lenses: the price path and its event features. Rare states keep every path, and no state weighs less than half an equal share in the grouping.
3. **Validate chronologically.** Expanding walk-forward inside 2020–2025: train 2020–22, test 2023; train 2020–23, test 2024; train 2020–24, test 2025. Every method choice, including how many families, uses these years only.
4. **Build the atlas.** Distributions, probabilities, timing, transitions and state × regime interactions per cell, against three baselines and two random-path versions. Sparse cells also get pooled estimates, always labelled as pooled.
5. **Confirm once on 2026**, untouched until the freeze, then review. Phase 2 starts only after your review.

Decided on 25 Oct: a separate tool that reads tradelab, T0 from 1 Jan 2020 to 15 Sep 2026, and the nine changes in the next section. First full run on your PC: about 3–4 hours (an estimate, not measured).

## The nine changes (25 Oct revision)

Seven changes are taken as proposed (one adjusted) and two with a different method; two of them fix flaws in the first design.

| # | change | decision | how it is built |
| --- | --- | --- | --- |
| 1 | expanding walk-forward; 2026 untouched | adopted, adjusted | Folds: train 2020–22 → test 2023; 2020–23 → 2024; 2020–24 → 2025. Families are finally fitted on 2020–2025 and frozen; 2026 is checked once. A finding is validated on the folds; 2026 then marks it confirmed, consistent, contradicted or underpowered, because 2026 is too short to be a pass/fail gate for single cells. |
| 2 | audit rare states before training | adopted | A coverage stage runs before the freeze and reads nothing after T0 (stage table below). |
| 3 | state-aware learning | adopted, different weighting | Every path of a rare state stays in the grouping sample; only common states are thinned. Weights give each state at least 5.6% (half an equal share) of the fit. Not equal weights: a state with a few thousand paths would take over a family and fit its noise. Weights define families only; atlas numbers are never weighted. |
| 4 | richer path identification | adopted in part | Added: an event-feature lens (drawdown from the running high, run-up from the running low, when the high and low came, ±1 ATR order, choppiness, session-1 move, close), a 30-minute intraday grid, and declared rare-path events. Not adopted: grouping raw 625-bar paths or shape-warping. Bar noise swamps the distances and warping is quadratic in the number of paths; the 3-minute bars still feed every timing and event measure. |
| 5 | methods chosen inside development data | adopted, fixes a flaw | The first design chose the number of families partly by whether they reappeared in 2023–2026, the period that then confirmed cell results. Every choice now uses 2020–2025 only. |
| 6 | more realistic random paths | adopted in part | Two versions. Shared donor days: one random other date per session for all stocks of a T0 date, rescaled to T0 volatility; keeps market-wide co-movement and each day's gap and range. Shuffled order: the window's own five sessions in random order; keeps volatility clustering, gaps and correlation exactly and destroys only the order. Fresh days with realistic volatility clustering would need a fitted volatility model per stock: not in Phase 1. |
| 7 | equities apart from ETFs; survivorship | adopted, replaces "cash-like rule only" | The primary atlas is equities; ETFs get their own table. ETFs come from NSE's ETF list (eq\_etfseclist.csv) joined with a printed name rule (BEES, ETF or IETF endings), because NSE's list holds today's names only; without the list, the name rule alone, marked incomplete. Stocks that stop trading inside the cache are listed (a symbol change can show as one name stopping and another starting) and the atlas is rerun without them; stocks delisted before the cache existed have no bars, so that bias is stated, not measured. |
| 8 | state × regime interaction | adopted, fixes a gap | Per cell: (cell − same state, all regimes) − (same regime, all states − everything), with its interval, inside the false-discovery control. |
| 9 | rare behaviour, pooling | adopted | Declared rare paths (shakeout, failed breakout, delayed recovery, gap-and-fade, gap-and-go) with rates per cell; effective-sample thresholds; pooled estimates shrunk toward the state + regime prediction, shown beside the raw ones, labelled pooled, never used for a label. |

2026 (to 15 Sep) has 174 sessions: broad bearish 19 days, broad bullish 31, high stress 37, narrow bullish 36, mixed 51 (regime labels only; no outcome looked at). It can confirm the margins and the headline tests; most single cells can only be called consistent or contradicted there.

Three corrections came from testing pathlab on synthetic data before this review and stay in. A valley test for families: stability alone accepted slices of a pure random walk as families. Path regions when nothing separates. Minimum support per window: 200 dates in each period would have made mixed, broad bearish and high stress untestable in 2020–2022.

## Data audit: what exists

Everything Phase 1 needs exists on your PC except AMFI's size lists; the 3-minute files are about three weeks behind. Figures below come from your tl\_check and export runs of 25 Oct.

| source | what it holds | span | Phase 1 use | status |
| --- | --- | --- | --- | --- |
| Daily bars, `SYMBOL_daily.parquet` | Kite Connect OHLCV on one consistent corporate-action basis; 2,495 equity files, a median 1,502 trading each session | from 2015-01-01 (fixed anchor) to the last run (APOLLOPIPE: 2026-10-23) | states, daily paths | ready |
| Index files: NIFTY50, NIFTYMIDCAP150, NIFTYSMLCAP250, INDIAVIX | daily closes | same | market regime | ready |
| Breadth, tradelab's saved copy | advancers ÷ (advancers + decliners) over the cached stocks | built 24 Oct 22:07 | market regime | ready |
| Panel, `panel_oc\panel.parquet` | 1,902,513 rows × 322 columns; a stock-day is in only if its 60-session median turnover is at least ₹1 crore (point in time); 241 usable features | 2016–2026 | the eligible universe on each day; features wait for Phase 2 | ready |
| 3-minute bars, `intraday_3m\SYMBOL_3m.parquet` | 1,408 files; 125 bars a session, stamped with their start time (09:15, 09:18 … 15:27, IST); 100% of APOLLOPIPE's bars on that grid | 2021-10-01 to 2026-10-06 | intraday paths | ready, 3 weeks behind |
| pick\_meta run PM\_20261017\_001 | D and ENS ranks 1–10, 33,100 picks, hash checked | 2020-01-16 to 2026-09-15 | picks vs universe | ready |
| AMFI half-yearly size lists | none in `sectors\amfi\` | — | market-cap segments | missing |
| In this workspace | your export TLX\_20261025\_001: picks only, no day-by-day file, no universe, no 3-minute bars | 2020–2026 | a cross-check of the picks' paths | here |

Code to reuse, read only: tradelab's readers (stage E's bar loader and lower-circuit rule, the 3-minute reader and its usable-session test, the panel's universe), tl\_state for states and tl\_context for regimes. Both were tested point in time; production files are never written.

Earlier path research covered picks or a small sample, never the universe by state and regime:

- Barrier scans (21 Sep, 250 symbols, no liquidity floor): +1.5 ATR was touched before −1.0 ATR 27.7–28.5% of the time, a median of 3 sessions in.
- Trade forensics (TF\_20261019\_001, D's top 3): 39.5% of trades fell 5% below entry at some point; 19.1% of those that fell first recovered to +5%.
- The recovery study and recovery check: fallen top-3 trades only.

## Gaps and limitations

The two that shape the design most: intraday history is half as long as daily, and the universe holds only stocks that were still listed when the cache was built. Each limit is measured and printed in the report, not assumed away.

| limit | what it does to the results | how Phase 1 handles it |
| --- | --- | --- |
| 3-minute bars start 2021-10-01 | intraday findings rest on about 5 years, daily ones on about 6¾ | separate intraday atlas with its own walk-forward: train from 2021-10 to the end of 2022, test 2023; to 2023, test 2024; to 2024, test 2025; 2026 untouched |
| 1,408 intraday files against roughly 2,000 panel stocks | some stock-days have daily paths only | the audit measures coverage by year, liquidity and state, and the report says whether covered stocks differ from the rest |
| 3-minute cache last bar 6 Oct | nothing after early October intraday | the atlas stops at T0 = 15 Sep 2026 anyway (below), so a refresh is optional |
| 3-minute resolution, first bar 09:15 | two events inside one 3-minute bar cannot be ordered; nothing from the pre-open auction | ties are flagged, never guessed |
| no AMFI lists | no market-cap segments | liquidity tiers (turnover at T0) stand in, labelled as liquidity, not size; real segments once you add the lists |
| ETFs in the cache | money-market funds look like perfectly calm stocks (doc 23: 16% of D's top-3 picks in 2026) | the primary atlas is equities only; ETFs (NSE's ETF list joined with a printed name rule, or the name rule alone, marked incomplete, until the list is added) get their own table; cash-like funds are also caught by the 0.2% range rule; every exclusion is counted |
| survivorship | stocks delisted before the cache was built are missing, so paths that ended in delisting are under-represented (likely fewer persistent declines; an inference) | the coverage audit lists stocks that stop trading inside the cache (a symbol change can show as one name stopping and another starting) and the atlas is rerun without them; delistings before the cache existed are stated, not measurable |
| corporate actions | a split inside a window would look like a crash | windows crossing a corporate-action suspect (the cache's `.ok.json`) are flagged and left out of families; a 3-minute session more than 0.5% off its daily bar is unusable |
| circuit locks | a locked session has no tradeable path | a session whose high equals its low is flagged (an up-lock or a down-lock by its close), kept as its own marker |
| breadth counts cached survivors | not NSE's official advance/decline | stated; the regime labels already carry this |
| states and regimes are research defaults (declared 24–25 Oct) | they may carry little information about what comes next | that is exactly what the atlas tests; nothing assumes they work |
| the forward paper test (from 30 Sep) | using its days would spend the only untouched evidence | no T0 after 15 Sep 2026; every window ends before the paper test began |
| the 2026 final test is short | 174 sessions; four of the five regimes have under 40 days | it confirms the margins and headline tests; single cells are marked confirmed, consistent, contradicted or underpowered |

## Definitions

One observation is one eligible stock on one session, classified at that session's close (T0) and followed for the next five sessions. The terms below are fixed before any run.

| term | definition |
| --- | --- |
| T0 | the 15:30 IST close of session d. Everything that classifies a stock-day is known at T0. |
| eligible universe | stock-days with a row in `panel_oc` (60-session median turnover at least ₹1 crore, point in time, quality gate passed), minus cash-like funds (60-session median daily range under 0.2% of price at T0), minus ETFs (counted in their own table), minus stock-days with under 60 sessions of history. T0 from 1 Jan 2020 to 15 Sep 2026. |
| stock state at T0 | tl\_state v1.1's primary state from the stock's own daily bars up to T0: S01, S02, S03, S06, S07, S08, S09, S12 or UNKNOWN (9 groups). INSUFFICIENT\_DATA is counted, not analysed. Phase, events and S05 appear only in secondary tables. |
| market regime at T0 | tl\_context's label at the close of d: broad bullish, narrow bullish, mixed, broad bearish, or high stress (the stress flag overrides direction). 5 groups. |
| cell | one state × one regime: 45 cells, plus margins (each state over all regimes, each regime over all states, everything). |
| five-day window | the next five NSE sessions on the market calendar (NIFTY's sessions): session 1 = d+1 … session 5 = d+5. BigMove's trade, session-1 open to session-5 close, sits exactly inside it. |
| reference prices | C0 = the T0 close, so the opening gap is part of the path. O1 = the session-1 open, the first price anyone can trade. Every measure is given from C0; trade measures also from O1. |
| units | percent of C0, and ATR units: multiples of the stock's 14-day average daily range at T0 (Wilder ATR ÷ C0). One ATR is an ordinary day's range for that stock, so a calm stock and a wild one become comparable. |
| incomplete window | no bar on one of the five market sessions (suspension, delisting). Kept and counted as its own outcome, never dropped silently. |
| BigMove selections | D and ENS ranks 1–3 (the traded picks) and ranks 1–10 from PM\_20261017\_001. Universe tables include every eligible stock; picks get their own tables and a same-day, same-cell comparison with the stocks not picked. |

&#91;embedded content: one observation · what classifies it and what is measured\]

The left side decides which cell a stock-day falls in; the right side is what the atlas measures. Nothing on the right ever moves a stock-day between cells.

## The path dataset

Every eligible stock-day gets one row with its whole five-day journey: over a million rows from daily bars (2020 to 15 Sep 2026), plus 3-minute detail on the rows whose five sessions all have usable 3-minute bars. Values are kept both in percent and in ATR units.

| group | what is measured | from |
| --- | --- | --- |
| opening | the gap (session-1 open vs C0) | daily bars |
| opening | first 15, 30 and 60 minutes: move and range; which way the open drove; a rejection (the first hour's extreme given back by 0.5 ATR or more before 10:15) | 3-minute bars |
| each session, 1 to 5 | open to high, open to low, open to close, close to close, the day's range, where the close sits in that range, volume against its 50-session median at T0 | daily bars |
| each session, 1 to 5 | time of the day's high and of its low, which came first, close against the session's VWAP, the last hour's move | 3-minute bars |
| the whole window | the 5-day return from C0, and from O1 (BigMove's trade before costs) | daily bars |
| the whole window | MFE and MAE (best high, worst low) from C0 and from O1, with the session they happened in | daily bars |
| the whole window | the minute MFE and MAE happened | 3-minute bars |
| thresholds | first touch of ±0.5, ±1, ±2, ±3 ATR and of +3%, +5%, +10%, −3%, −5%, −8%: when, and which came first. In daily bars a same-session tie is flagged; 3-minute bars resolve it where usable | both |
| recovery | touched −1 ATR (or −5%) first and still ended above C0; back to C0 after −1 ATR; on to +1 ATR after −1 ATR | both |
| momentum and reversal | the signs of the five close-to-close moves (32 patterns); the session of the window's highest and lowest close | daily bars |
| volatility | range in sessions 4–5 against sessions 1–2; realised volatility in the window against the 20 sessions before T0 | daily bars |
| market during the window | NIFTY, Midcap 150 and Smallcap 250 over the same sessions; the regime label at each session's close | daily bars |
| state during the window | tl\_state's state at each session's close, observed after T0 (describes, never classifies) | daily bars |
| flags | incomplete window, corporate-action suspect, lower or upper circuit lock, how many of the five sessions have usable 3-minute bars | both |
| shape and events | drawdown from the running high, run-up from the running low, path length (the sum of the absolute session moves), direction changes between sessions: the event path below | both |
| rare paths | shakeout (−1 ATR came first, ended above +0.5 ATR); failed breakout (+1 ATR came first, ended below C0); delayed recovery (fell to −1 ATR, back above C0 only in session 4 or 5); gap-and-fade (gap up 0.5 ATR or more, session 1 closed 0.5 ATR or more below its open); gap-and-go (the same gap, session 1 closed above its open) | both |

Three fixed-length vectors feed the family search in the next section:

- **Daily path:** the session-1 open, then the high, low and close of sessions 1–5, each from C0 in ATR units (16 numbers).
- **Intraday path:** the cumulative move from C0 in ATR units at 14 points a session (the 09:15 open, every 30 minutes from 09:45 to 15:15, and the close), over five sessions (70 numbers). Exact timings still come from all 125 bars.
- **Event path:** 12 numbers per window: the gap, the session-1 close, the 5-day close, the window's high and low (ATR from C0) and when each came (0 to 1 across the window), the drawdown from the running high, the run-up from the running low, the path length, the ±1 ATR order (+1, −1 or 0) and the number of direction changes. Scaled by the median and interquartile range of the training window. Daily and 3-minute versions.

## Discovering path families

Families come from development data only (2020–2025) and must pass three tests: stable under date-block resampling, reproduced in every walk-forward test year, and separated from their neighbours by a valley in the density. If no number of groups passes, the paths form a continuum: they are described as path regions, or by the measures alone, and never forced into families.

1. **Two lenses.** The price path: the daily 16 numbers and the 3-minute 70. The event path: the 12 event numbers (previous section). Price paths are capped at ±8 ATR for grouping only, and principal components keep 95% of the variation; event numbers are scaled by their median and interquartile range. Each lens goes through the same tests on its own.
2. **A state-aware sample.** Up to 300,000 paths per training window, balanced by year. Every path of a rare state is kept and only common states are thinned; weights give each state at least half an equal share (5.6%) of the fit. The weights only define the families; every atlas number stays unweighted.
3. **How many families (declared).** For k = 4 to 16 with k-means (pathlab's own seeded code, identical across library versions). Stable: mean adjusted Rand index of at least 0.75 over 20 date-block refits of the 2020–2025 sample. Reproduced: in each fold, the test year's groups pair with the training groups within half a group's spread. Separated: between every group and its nearest neighbour, the density along the line joining them dips to at most half the lower peak. Families are the largest k passing all three; path regions are the largest k that is stable and reproduced; neither means no stable structure.
4. **Why the valley test.** On synthetic random walks, k-means found stable, reproducible groups; on six planted families, the stable-and-reproduced rule split the largest one in two. Only the valley told real families from slices of one cloud (tested; a two-peak score was tried first and failed on flat-topped clouds).
5. **A second method as a check.** A Gaussian mixture, started from the k-means solution; a family that only one method finds is marked.
6. **Minority behaviours.** A small family stays when it passes the three tests. Inside each family, two peaks in the 5-day close are checked in the training and the test windows; a stable second peak is reported as a sub-family.
7. **Two random-path versions.** Shared donor days: each session replaced by the same random other date for every stock of that T0 date, rescaled to the T0 volatility; this keeps market-wide co-movement and each day's gap and range. Shuffled order: the window's own five sessions in random order; this keeps the real days exactly (volatility clustering, gaps, correlation) and destroys only their order. A family is real structure only where its share differs from both versions, with the interval to show it.
8. **Names.** Each family is named from its centre by declared rules (for example, immediate expansion: up at least 0.5 ATR by the session-1 close and higher still at session 5). The name describes the centre; the data decides membership.

## The conditional atlas

For each of the 45 cells, and for each state and each regime on its own, the atlas reports the family odds and the full path measures against three baselines and the state × regime interaction. Every number carries an interval built for overlapping windows, and a false-discovery control covers the whole run.

**What each cell reports**

- *Support:* observations, distinct T0 dates, distinct stocks, and the effective sample size (what the cell is worth in independent observations, taken from the bootstrap).
- *Family odds:* the probability of each family, with its 95% interval.
- *Distributions,* overall and inside each family: the 5-day return (5th, 10th, 25th, 50th, 75th, 90th and 95th percentiles, and the mean), MFE and MAE, and when each happened (session; minute where 3-minute bars exist).
- *Excursions:* the chance of reaching +1, +2 and +3 ATR, and of falling to −1, −2 and −3 ATR (and the same at +5%, −8% and the other fixed thresholds).
- *Order:* the chance that +1 ATR comes before −1 ATR, +2 before −1, +1 before −2, and the median time to each.
- *Turning points:* on which session, and at what time of day, the window's high and low fell.
- *Recovery:* after −1 ATR came first, the chance of getting back to C0 and of going on to +1 ATR; after −2 ATR, the chance of still ending above C0.
- *Continuation and reversal:* after a session-1 close at least 0.5 ATR up, the chance that session 5 closes higher still, and the chance it ends below C0; the mirror for a weak session 1.

**Baselines and the interaction.** Each cell is compared with (a) every eligible stock-day, (b) the same regime with all states, which isolates what the state adds, and (c) the same state in all regimes, which isolates what the regime adds. (d) The interaction asks whether the combination changes the odds beyond what the state and the regime do apart: (cell − same state, all regimes) − (same regime, all states − everything), with its own interval. Families are also compared with both random-path versions.

**The headline test: do state and regime tell us anything?** One number for the whole atlas: how much knowing the state narrows the family odds (mutual information), against the same number after shuffling states among the stocks of the same day. Shuffling within a day keeps that day's market, so only the state's own information is tested. The regime gets the same test against regime labels shifted in time, which keeps their run lengths.

**Overlapping windows.** Consecutive T0s of one stock share four of five days, and stocks on the same date move together. So intervals come from a circular block bootstrap over T0 dates: 10 consecutive sessions a block, all stocks of a drawn date kept together, 2,000 resamples, seed 7 (the project's convention). A second check keeps only every fifth T0 per stock, so no windows overlap; its figures must fall inside the main intervals.

**Many comparisons.** Families alone give roughly 45 cells × 10 families × 4 comparisons ≈ 1,800 tests per window, plus the headline path measures. A Benjamini–Hochberg false-discovery rate of 5% applies across every test of a window, and the report prints the count of tests and how many false finds to expect.

**Minimum support (declared after the coverage audit).** A cell is described over 2020–2026 with at least 200 dates and 1,000 paths (intraday: 100 and 300). It can be labelled only where each window used has at least 50 dates, 250 paths and 8 separate 10-session stretches (intraday: 30, 150 and 6), and an effective sample from the coverage audit's T0-known measure. These numbers are proposals until the coverage counts are in. Below them a cell shows counts only, marked *insufficient*.

**Pooled estimates for sparse cells.** Beside each raw estimate: the cell shrunk toward its state + regime prediction, more when its effective sample is small (empirical Bayes). Always labelled pooled; never used for a label or counted as independent evidence.

**Rare paths.** Shakeout, failed breakout, delayed recovery, gap-and-fade and gap-and-go rates for every cell, with intervals, so uncommon paths are reported even when no family holds them.

**Segments.** Equities in the main atlas, ETFs in their own table. Year by year for every supported cell. Liquidity tiers (turnover at T0, point in time) stand in for market cap until the AMFI lists arrive. Survivorship sensitivity: the atlas rerun without the stocks that stop trading inside the cache.

## Transitions inside the window

Three kinds of change are described: the stock's state, the behaviour stage of each session, and the market regime. All are observed after T0, so they explain what happened and when; none is used to classify, and no trigger is searched for in Phase 1.

**1. State changes.** tl\_state gives a state at each session's close (each from bars up to that close). The atlas reports where each T0 state ends up by session 5, per regime, as a transition table; the session of the first confirmed change (tl\_state needs two sessions to confirm one, so session 2 at the earliest); and the commonest sequences with repeats collapsed (for example S09 → S07 → S02).

**2. Behaviour stages.** Each session gets one stage label from its close-to-close move and its range, both in ATR, by a declared rule:

| stage | rule |
| --- | --- |
| Up | close to close +0.5 ATR or more |
| Down | close to close −0.5 ATR or less |
| Quiet | in between, range under 0.8 ATR (compression) |
| Churn | in between, range 1.5 ATR or more (whipsaw) |
| Flat | in between, anything else |

Where 3-minute bars exist, each session is also split into morning (09:15–12:15) and afternoon (12:15–15:30) with the same rule at half the thresholds, so “fell all morning, recovered in the afternoon” is visible. Named sequences are then counted, with when they start:

- decline → stabilisation → recovery: Down, then Quiet or Flat, then Up;
- compression → expansion: two Quiet sessions, then Up or Down with a range of 1.5 ATR or more;
- breakout → rejection → continuation: Up, Down, Up;
- recovery → exhaustion → reversal: Up, Quiet or Flat, Down.

A sequence is called a behaviour only if it occurs more often than in both random-path versions; otherwise it is reported as what chance produces.

**3. Regime changes.** The regime label at each session's close splits the windows into regime held, improved (towards broad bullish) and worsened (towards broad bearish or high stress), with the session it changed. Paths are compared across those groups.

## Validation

A finding is **validated** when it passes the 5% false-discovery control in the first training window where its cell has enough data, and its 95% interval excludes zero in the same direction on the later development years. 2026, untouched until then, marks it confirmed, consistent, contradicted or underpowered. Every method choice uses 2020–2025 only.

| label | when it is used |
| --- | --- |
| validated | passes the false-discovery control in its first supported training window (2020–22, 2020–23 or 2020–24); on the later development years pooled, the 95% interval excludes zero in the same direction; not unstable |
|  confirmed | validated, and 2026 agrees with its interval excluding zero |
|  consistent | validated; 2026 points the same way but its interval spans zero (2026 is short) |
|  contradicted | validated; 2026 points the other way with its interval excluding zero |
|  underpowered | validated; below the minimum support in 2026 |
| hypothesis | passes in a training window, not on the later years |
| inconclusive | enough data, passes nowhere |
| insufficient | never enough data in a training window and its later years |
| unstable | flips sign in more than a third of its supported years, or contradicts itself across windows |

**Before it reaches your data.** Synthetic worlds with known answers, every one required to pass: planted families at known shares (the search must find exactly them, named right), planted state and regime effects, a planted interaction, no effect at all, and pure random walks (no families, no information, no verified cells). Also: every bar after T0 scrambled leaves every T0 value unchanged; a run killed halfway resumes to identical outputs; the coverage audit reads nothing after T0; your library set and the newer one give identical results.

**On your data, every run:**

- Coverage audit before the freeze; the fold boundaries and minimums are set from its counts, never from outcomes.
- Expanding walk-forward with a 5-session gap at each boundary; 2026 checked once, after the freeze.
- Overlap: intervals from the date-block bootstrap, plus the every-fifth-T0 check.
- Both random-path versions go through the same code; what state or regime information survives there comes from which stocks sit in a state, not from timing.
- Integrity: the picks' paths must match TLX\_20261025\_001 on at least 99.5% of picks, or the run stops.
- Survivorship sensitivity: the atlas rerun without the stocks that stop trading inside the cache.

**Picks against the universe.** D's and ENS's top 3 and top 10 get their own tables, against the stocks not picked on the same day in the same cell. Descriptive only. The 2026 part is not untouched: picks by state through September 2026 were examined on 25 Oct (doc 23).

## Outputs and how it runs

A separate tool, `pathlab`, in its own folder beside tradelab: it imports tradelab's readers and its state and regime code, changes nothing in tradelab, and writes only under `C:\QuantData\pathlab\`. Every stage saves its work, so a stopped run resumes where it stopped. The settings live in `PATHLAB_DECLARATION.json`. `freeze` records its fingerprint after you review the coverage; families, atlas and report then refuse any other declaration, and a change needs a written reason that every report prints.

| stage | what it does | saved to |
| --- | --- | --- |
| audit | reads everything, prints what exists, stops if something essential is missing | `PL_…\audit\` |
| coverage | outcome-blind counts per state × regime × year and window; the ETF and survivorship lists | `coverage\COV_…\` |
| freeze | records the declaration's fingerprint once you have reviewed the coverage | the pathlab ledger |
| build | per stock: states, daily and 3-minute path rows, both random-path versions; then the integrity check | `pathlab\cache\`, one file per stock |
| families | both lenses, the walk-forward choice of k, the valley test, the mixture check, assignments | `PL_…\families\` |
| atlas | cells, baselines, interactions, pooled estimates, false-discovery control, labels, transitions, picks | `PL_…\atlas\` |
| report | `ATLAS.md`, `ATLAS.html` with charts, `summary.json`, `outputs.sha256`, a line in the run log | `PL_…\` |

Released now: audit and coverage only; the other stages are refused until the freeze. The commands:

```
call C:\Users\karanvsi\PyCharmMiscProject\pathlab\env_pathlab.bat
python tests\test_pathlab.py quick
python pathlab.py audit --root %CACHE_DAILY_ROOT%
python pathlab.py coverage --root %CACHE_DAILY_ROOT%
```

Expected cost on your PC (an estimate, not measured): coverage 5–10 minutes. The full run after the freeze takes about 3–4 hours the first time, minutes for a later stage alone, and 2–4 GB of disk.

## Decisions taken (25 Oct)

You approved the protocol on 25 Oct with three changes in the morning and nine more in the afternoon (above). Comment anywhere before the freeze to change a definition, a threshold or a test; after it they are fixed.

| decision | answer |
| --- | --- |
| packaging | a separate tool, `pathlab`, that reads tradelab's code; no new tradelab version |
| funds and ETFs | ETFs in their own table, out of the main atlas (replaces the morning's "cash-like rule only"); cash-like funds are also caught by the range rule |
| span | T0 from 1 Jan 2020 to 15 Sep 2026 |
| training and testing | expanding walk-forward inside 2020–2025, 2026 untouched until the freeze (replaces "discovery 2020–2022, confirmation 2023–2026") |
| market-cap segments | liquidity tiers until AMFI's lists are added |
| next step | the coverage audit on your PC; fold boundaries and minimums from its counts; then the freeze |

The 3-minute files need no refresh: they run to 6 Oct, and the last window ends on 22 Sep.

**Remaining build.** About 4–6 hours on my side for the revision (two lenses, the walk-forward, the second random version, interactions, pooling, rare paths, the freeze), each with its synthetic test. Then the first full run on your PC, about 3–4 hours (estimates, not measured). Phase 1 ends only when you have reviewed the atlas; Phase 2 does not start before that.
