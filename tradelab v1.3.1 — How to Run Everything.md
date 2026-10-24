# tradelab v1.3.1 — How to Run Everything

Oct 25, 2026 · @karan

One double-click a night (`run_tonight.bat`) runs Run Now, pick\_meta and the watch page. The export, the backtest and the simulator are one command each, typed in a cmd window.

## Before you start

Install once and check it. After that, every cmd window starts with one setup line.

1. Unzip `tradelab_v1.3.1.zip` beside `bigmove_deploy`, replacing the whole folder `C:\Users\karanvsi\PyCharmMiscProject\tradelab`. Copy every file. Nothing in `bigmove_deploy` changes, and your outputs in `C:\QuantData\tradelab` are kept.
2. Open cmd and run the setup line (the first line below).
3. Run the test. It must end `VERIFIED (tradelab v1.3.1)` and takes about 10–16 minutes. On your PC it also runs a copy of `run_tonight.bat` against stand-in folders in its temporary folder, never against `bigmove_deploy`.
4. Run the preflight. It must end `CHECK OK`. Read its `tonight (v1.3.1)` and `watch (v1.3)` lines.

```
call C:\Users\karanvsi\PyCharmMiscProject\tradelab\env_tradelab.bat
python tests\test_tradelab.py
python tl_check.py --root %CACHE_DAILY_ROOT% --pm-run PM_20261017_001
```

**Every new cmd window starts with the setup line.** It prints `tradelab ready`, moves you into the tradelab folder and sets `%CACHE_DAILY_ROOT%`.

```
call C:\Users\karanvsi\PyCharmMiscProject\tradelab\env_tradelab.bat
```

## Every night: run\_tonight.bat

Double-click `run_tonight.bat` in the tradelab folder when you would press Run Now. It replaces Run Now, `run_pick_meta.bat` and `run_watch.bat`, in this order:

1. **Run Now** — `bigmove_deploy\run_now.bat`, unchanged, started from its own folder. If it asks for a key at its end, press it once; the rest carries on.
2. **pick\_meta** — publishes tonight's cards plus any recorded day still without one (`publish --all-missing`; a late card is marked LATE, as before), then scores the tracker.
3. **The watch page** — builds tonight's page plus a page for any recorded day still without one (`tl_watch --all-missing`), then opens tonight's page.

&#91;embedded content: the nightly chain · 3 steps and the files each one writes and reads\]

A step that fails leaves its files as they were, so the next step finds nothing new instead of something wrong.

**Every step runs even if an earlier one failed.** Each step only reads what is already recorded, so a failed Run Now gives you nothing new, never a wrong card or page.

**The last line tells you how it went:** `RESULT: OK`, or `RESULT: FAILED at [step 1 Run Now: code 1]` (each failed step with its code). Each night also adds one line to `C:\QuantData\tradelab\tonight_log.txt`: the date, the time and each step's exit code.

**Finish before 09:00 of the next weekday.** A page written later is marked LATE and is not forward evidence; pick\_meta's cards count as on time only when published before the next session's open.

**Optional, no click at all:** Windows Task Scheduler can run `run_tonight.bat auto` (no pause, no browser). Only use it if Run Now finishes without you; a Kite login prompt would fail at once. Pick the time you usually press Run Now, and do not keep a second task that runs `daily_run.bat` at the same time.

```
schtasks /Create /TN "tradelab_tonight" /TR "C:\Users\karanvsi\PyCharmMiscProject\tradelab\run_tonight.bat auto" /SC WEEKLY /D MON,TUE,WED,THU,FRI /ST 20:45
```

## All trades with the updated data

One export run gives every D and ENS pick, ranks 1–10, from 16 Jan 2020 to 15 Sep 2026, day by day, with the market and the stock's state at each pick. Run it once now; it is the slowest of the commands here.

```
python tl_export.py --root %CACHE_DAILY_ROOT% --pm-run PM_20261017_001
```

The output lands in the newest folder of `C:\QuantData\tradelab\export\`, named like `TLX_20261025_001`. Open these in order:

1. `report.md` — the summary and the reconcile line (the plan reproduced trade by trade).
2. `research\research_report.md` — section 6 is D and ENS by the stock's state and its phase (early, mature, deteriorating). Descriptive only: about 50 cells per engine, so 2–3 look different by luck.
3. `trades.csv` — one row per pick; the state at the signal-day close is in the `pre_state_*` columns.
4. `trade_days.csv` — each pick day by day, sessions 0–5, with the market that day.
5. `cards\cards_sample.html` — the best and worst trades told as a chain, each with its state line.
6. `DATA_DICTIONARY.md` — what every column means and when it is known.

**If it ends with `STOPPED`,** the plan did not reproduce pick\_meta's record and nothing else was written. Send `STOPPED.txt` and `reconcile_mismatches.csv`.

**Paper trades since 30 Sep are not in this run,** because pick\_meta's research run ends on 15 Sep. They are on the watch pages and in `watch_all.csv` (next section).

Useful options: `--engines D` · `--ranks 1-3` (only the traded picks) · `--from 2023-01-01 --to 2026-09-15` · `--cards 20` (more sample cards).

## Watchlist pages

One command builds a page for every recorded paper day that has none yet, then opens the newest. Use it now to get the pages since 30 Sep; after that, `run_tonight.bat` does it every night.

```
run_watch.bat --all-missing
```

Pages built after their time are marked LATE: they show each day exactly as it was at that close, but they are not forward evidence.

To rebuild one day (for example after its pick\_meta card was published late):

```
python tl_watch.py --root %CACHE_DAILY_ROOT% --date 2026-10-23 --force
```

**Where they are:** `C:\QuantData\tradelab\watch\WATCH_YYYYMMDD.html` (the page), `.md` (text) and `.csv` (every number). `watch_all.csv` holds every page's rows in one table, with when each page was written and whether it was LATE.

**How to read a page, top to bottom:**

1. **The market banner** — the environment at the close (Broad bullish, Narrow bullish, Mixed, Broad bearish, or High stress), how long it has lasted, NIFTY vs Midcap and Smallcap, breadth, VIX, and the declared warning signs on that day.
2. **D's and ENS's top 10** — per stock: size class and the index it moves with, its state chip (state, confidence, sessions in it, phase), the latest event and risk flags, pick\_meta's P(+5%), P(+10%), P(−8%) and P(stuck), and 20-day strength against its size index. Ranks 1–3 in bold are the paper test's trades.
3. **A card per stock** — why it is in its state (every condition, ticked or not), what would end it with prices, its last transitions and 20 sessions, the market around it, the measurements, and the Soul card copied unchanged.

Nothing on the page filters or re-ranks a pick. Expect a fair share of UNKNOWN: when the evidence conflicts, the engine says so instead of forcing a label.

## Backtest

A backtest runs a different trade rule on the same picks as the plan and says BETTER, WORSE or NOT SHOWN, with a 95% interval. Write the pass rule before you run.

```
python tl_backtest.py --root %CACHE_DAILY_ROOT% --pm-run PM_20261017_001 --strategy strategies\open6.py --declare "PASS if the paired lower bound > 0"
```

This example sells at the 6th session's open instead of the 5th close. Every strategy below trades D's ranks 1–3 (the paper test's trades) unless you change it.

| strategy file | what it tests |
| --- | --- |
| `open6.py` | sell at the 6th session's open instead of the 5th close |
| `half_target_stop.py` | sell half at +5%, stop at −8%, the rest at the 5th close |
| `trail_after_5pct.py` | trail 4% under the highest high once the trade is up 5%, hold up to 10 sessions |
| `scale_20_80_vwap.py` | 20% at the open, 80% when a 3-minute close crosses above VWAP |
| `orb_add_on.py` | 50% at the open, 50% on a break of the 15-minute opening-range high |
| `dip_or_fallback.py` | buy 1% under the open until 11:00, else buy at 11:00 |
| `filtered_rsi_entry.py` | skip picks with P(stuck) of 10% or more; buy only if RSI 14 at the signal close is under 70 |
| `weak_close_exit.py` | sell after 14:30 in sessions 2–4 if under VWAP and down 3% or more |
| `hypo_skip_smallcap_fall.py` | a hypothesis: skip picks made on a day Smallcap fell 1.5% or more |
| `plan.py`, `plan_intraday.py` | the paper test's own plan (daily / on 3-minute bars) |

| option | what it does |
| --- | --- |
| `--engines ENS` | the other engine (`--engines D ENS` for both) |
| `--ranks 1-10` | which ranks to trade |
| `--from 2023-01-01 --to 2026-09-15` | one period only |
| `--split-date 2024-01-01` | the comparison before and from that date; choose the date before you run |
| `--compare strategies\plan_intraday.py` | compare against another strategy instead of the plan |
| `--capital 1000000` | starting capital for the money view |
| `--no-portfolio` | skip the money view (faster) |
| `--tag "what I am trying"` | a note kept in the run log |
| `--list-names` | every value a rule can use |

**Reading the report** (`C:\QuantData\tradelab\backtest\TLB_...\report.md`):

1. **The verdict** — the strategy's mean net per pick, the plan's on the same picks, and the paired difference with its 95% interval.
2. **The trades** — fills, exits, MFE and MAE, by year, engine and rank.
3. **The money view** — CAGR, worst drawdown with its dates, Sharpe and exposure.
4. **Where it helps or hurts** — the paired difference by the market at the pick and by the stock's state (state, phase, confidence, event, S05, volatility). About 60 rows per engine, so expect about 3 to look different by luck. `trades.csv` has the state at every pick.

**Discipline.** Every run is logged before it starts, and the report says how many backtests you have run. With 20 variants on the same years, one will look BETTER by luck; a rule found here is a hypothesis until the forward paper test confirms it.

To see how the plan itself did by stock state, use section 6 of the export, not a backtest: `plan.py` against the plan only compares the plan with itself.

## Simulator

The simulator replays one trade bar by bar and writes its story, a timeline and a chart. Give it a trade id, or a stock and a day.

**A pick from the research run** — copy any `trade_id` from the export's `trades.csv`; the format is engine-signal day-rank:

```
python tl_sim.py --root %CACHE_DAILY_ROOT% --pm-run PM_20261017_001 --trade D-20240425-2
```

**A paper pick, or any stock on any day:**

```
python tl_sim.py --root %CACHE_DAILY_ROOT% --pm-run PM_20261017_001 --symbol RELIANCE --signal-day 2026-10-09 --engine D
```

This works once the pick's 5 sessions are over. A newer pick shows `pending - data ends before session 5`; ignore its 'never entered' line, which only means the trade is not finished yet.

**Two rules on the same trade** — add one `--strategy` per rule (the default is `plan.py`):

```
python tl_sim.py --root %CACHE_DAILY_ROOT% --pm-run PM_20261017_001 --trade D-20240425-2 --strategy strategies\open6.py --strategy strategies\plan.py
```

**What it writes** in `C:\QuantData\tradelab\sim\TLS_...\`:

1. `<trade>__<strategy>_story.md` — every buy and sell with its reason; the market the day before, on the signal day and on each day held, with the result split into a market part and a stock part; then the stock's state at the pick (what it rested on, what would end it) and a table of its state on every day held.
2. `<trade>__<strategy>_timeline.csv` — every value on every bar. Add more with `--values rvol,clv,D_rsi14`.
3. `<trade>_chart.html` — the stock with its entries and exits, and NIFTY, Midcap and Smallcap beside it.

The state at the pick (session 0) is known before entry. The later rows were seen after entry: they explain a trade, they never select one.

## Where everything is saved

tradelab writes only under `C:\QuantData\tradelab`; it reads production files and never changes them.

| what | folder | open first |
| --- | --- | --- |
| Nightly run log | `C:\QuantData\tradelab` | `tonight_log.txt` |
| Watch pages | `C:\QuantData\tradelab\watch` | `WATCH_YYYYMMDD.html`, `watch_all.csv` |
| Export (every pick) | `C:\QuantData\tradelab\export\TLX_YYYYMMDD_NNN` | `report.md`, `research\research_report.md`, `trades.csv` |
| Backtests | `C:\QuantData\tradelab\backtest\TLB_YYYYMMDD_NNN` | `report.md`, `trades.csv`, `context_breakdown.csv` |
| Simulations | `C:\QuantData\tradelab\sim\TLS_YYYYMMDD_NNN` | `*_story.md`, `*_chart.html` |
| Log of every run | `C:\QuantData\tradelab` | `tradelab_ledger.jsonl` |
| Market breadth (saved once) | `C:\QuantData\tradelab\context` | refreshed by the export and the watch page |
| Paper picks (read only) | `%CACHE_DAILY_ROOT%\paper` | `ledger.jsonl`, `watchlists\WL_YYYYMMDD.md` |
| pick\_meta cards (read only) | `%CACHE_DAILY_ROOT%\research2\pick_meta\forward` | `PM_cards_YYYYMMDD.csv` |
| AMFI size lists (you add them) | `%CACHE_DAILY_ROOT%\sectors\amfi` | one Excel file per half-year |

## If something goes wrong

Most messages name their own fix. When one does not, send the whole window's text.

| you see | it means | do this |
| --- | --- | --- |
| `NOT VERIFIED: the tests need ...` | a package the tests use is missing | `pip install pypdf pdfplumber openpyxl reportlab`, then run the test again |
| `NOT VERIFIED` with `FAIL` lines | a check failed | send the FAIL lines and the last 3 lines |
| `CHECK FAILED` | the preflight found a problem with your data or setup | send everything tl\_check printed |
| `tonight: Run Now's file not found` | `run_tonight.bat` cannot find `run_now.bat` | edit the line `set "TN_RUN_NOW=..."` at the top of `run_tonight.bat` to the file you double-click as Run Now |
| `tonight: Run Now hands its work to another window (START)` | Run Now returns before its work is done, so steps 2–3 could run too early | send `run_now.bat` before relying on `run_tonight.bat` |
| `RESULT: FAILED at [step 1 Run Now: code N]` | Run Now reported an error | read Run Now's own lines higher in the window; steps 2–3 still ran |
| `[step 2 pick_meta: code N]` | publishing or scoring failed | double-click `run_pick_meta.bat` to see the error, and send it |
| `[step 3 tl_watch: code 2]` | tonight's page could not be built | the line starting `ERROR:` just above says why; send it |
| `NOT BUILT - ...` for an earlier day | that one day was skipped | tonight's page was still built; send the line |
| a page with blank pick\_meta numbers | that day's card was not published when the page was built | publish it, then rebuild the day with `--date YYYY-MM-DD --force` |
| `already written: ...` | the page for that day exists | add `--force` to rebuild it |
| `STOPPED` (export) | the plan did not reproduce pick\_meta's record | send `STOPPED.txt` and `reconcile_mismatches.csv` |
| `trade ... not found` (simulator) | that trade id is not in the research run | copy one from `trades.csv`, or use `--symbol` with `--signal-day` |

## What to send back

Send these in this order; each one unlocks the next step.

1. The test's last line: `VERIFIED (tradelab v1.3.1)`, or the FAIL lines.
2. tl\_check's `tonight (v1.3.1)` and `watch (v1.3)` lines — they show what Run Now runs and how many Soul cards the page found.
3. The first night's `RESULT` line, and the page or a screenshot of it.
4. From the export: `research\research_report.md` and `research\state_book.csv` — the first look at how D and ENS did by stock state and phase (in-sample, descriptive).
5. Any backtest `report.md` you want read, with the `--declare` rule you wrote before running it.
