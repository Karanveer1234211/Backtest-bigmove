# MANIFEST - tradelab v1.3.1

Built 2026-10-25. Added the same evening: tl_journal.py (tl_journal v1.0), an add-on - every other file below is unchanged and tradelab is still v1.3.1. Folder: C:\Users\karanvsi\PyCharmMiscProject\tradelab (beside bigmove_deploy, never inside it). Outputs: C:\QuantData\tradelab. Production files are read, never written. 'tradelab v1' = unchanged since the v1 bundle (a6cbcd209f3bc759); v1.1 = unchanged since the v1.1 bundle (14584256599dd3aa); v1.2 = unchanged since the v1.2 bundle (9352fdf46e7cdfcb); v1.2.1 = unchanged since the v1.2.1 bundle (48afedf09a571524); v1.3 = unchanged since the v1.3 bundle (f5e3b6149876888c); v1.3.1 = new or changed in v1.3.1.

| file | version | sha256 (first 16) | role |
|---|---|---|---|
| README_TRADELAB.md | tradelab v1.3.1 | 14468d38450b882d | how to install, run, read the watch page, the market layer and the stock states, write strategies; changelog |
| STATE_RULES.md | tradelab v1.3.1 | 2d0a68f0b1c5144e | v1.3: every stock state's rules, the precedence, confidence, phase, events, risk flags and settings (written by tl_state.py rules) |
| env_tradelab.bat | tradelab v1 | fc1a395d0e460583 | start a session: env_daily.bat, PYTHONPATH (tradelab, open_close, bigmove_deploy), TRADELAB_OUT (CRLF) |
| plugins/example_plugin.py | tradelab v1 | 52b7f8a1462d4a50 | example plugin: vol_surge, range_pos20, above_for() |
| run_tonight.bat | tradelab v1.3.1 | 02297436f887f90b | v1.3.1: EVERY NIGHT, one double-click - Run Now (unchanged, child cmd), pick_meta publish --all-missing + score, tl_watch --all-missing; every step reported; 'auto' for Task Scheduler (CRLF) |
| run_watch.bat | tradelab v1.3.1 | db63eec3f428871c | the watch page alone, by hand (arguments pass through, e.g. --date ... --force); opens the newest day's page (CRLF) |
| strategies/dip_or_fallback.py | tradelab v1 | fce97d839c256931 | example strategy |
| strategies/filtered_rsi_entry.py | tradelab v1 | af30c8ad09315f78 | example strategy |
| strategies/half_target_stop.py | tradelab v1 | 8f5ebeb98f88cf03 | example strategy |
| strategies/hypo_skip_smallcap_fall.py | tradelab v1.1 | e0f3691f06adfaed | example HYPOTHESIS (not a rule): the plan skipping W6 picks; run with --split-date |
| strategies/open6.py | tradelab v1 | 86a67db320a11d6f | example strategy |
| strategies/orb_add_on.py | tradelab v1 | e7ac4b38b71d56da | example strategy |
| strategies/plan.py | tradelab v1 | 98481fc8789d03e3 | example strategy |
| strategies/plan_intraday.py | tradelab v1 | fca2e77d51d4af68 | example strategy |
| strategies/scale_20_80_vwap.py | tradelab v1 | 629d2b225be2814a | example strategy |
| strategies/trail_after_5pct.py | tradelab v1 | cc08706cd7df6fd6 | example strategy |
| strategies/weak_close_exit.py | tradelab v1 | 64f072d7314c6535 | example strategy |
| tests/test_context.py | tradelab v1.3.1 | b9b5bca3fc861767 | sections 10-13: the market layer by hand, point in time, research, integration |
| tests/test_state.py | tradelab v1.3.1 | cfd5fd260ab29bd4 | sections 14-16: stock states (planted / absent, rules, point in time), the watch page / export / research integration, v1.3.1 catch-up, run_tonight.bat (executed on Windows), states in backtest / sim |
| tests/test_tradelab.py | tradelab v1.3.1 | fbc4cfd70f8367f6 | the test suite, sections 1-16 (must end VERIFIED (tradelab v1.3.1)) |
| tests/tl_ctx_world.py | tradelab v1.1 | 887f935958821722 | synthetic worlds with planted size / index structure and AMFI lists (CSV / Excel / PDF) |
| tests/tl_state_world.py | tradelab v1.3 | 43af9c384800a6f1 | price paths with planted stock states (S01..S12, S04 / S11 / S10 events, S05) |
| tests/tl_world.py | tradelab v1 | d70cd0564be62ba0 | synthetic worlds + stand-in production modules for the tests |
| tl_backtest.py | tradelab v1.3.1 | f2ea2bdec9d880e1 | COMMAND: a strategy vs the plan on the same picks, paired 95% intervals, money view, by market environment and (v1.3.1) by stock state, --split-date, ledger |
| tl_cards.py | tradelab v1.3 | d71662e5573d0be5 | COMMAND: trade cards (known before entry / forecasts / day by day / verdict) and the market diary |
| tl_check.py | tradelab v1.3.1 | ccd42ef263360e56 | COMMAND: 30-second preflight on real data (reads only), incl. index files, breadth count, AMFI lists |
| tl_common.py | tradelab v1.3.1 | ad7e75715da7556a | versions, path guards (never writes into the cache or production), hashing, tables, ledger, block bootstrap |
| tl_context.py | tradelab v1.2.1 | c3e65f0fa44e2467 | COMMAND + library: the market layer - indices, divergence, breadth, VIX, AMFI size, beta / correlation, declared environment labels, pre_/post_ context |
| tl_data.py | tradelab v1.1 | 46ceff88fdadda7f | read-only production access: picks (pick_meta run, hash-checked), daily bars + lock rule (stage E), calendar, 3-minute sessions, panel features (label firewall) |
| tl_dictionary.py | tradelab v1.3.1 | 5341a26ca250030d | the data dictionary: every column's meaning, unit and when it is known |
| tl_engine.py | tradelab v1.1 | f0601965341a62b7 | strategies and the bar-by-bar trade engine (fills, intrabar policy, honest exits, path, timeline); market-context rule values |
| tl_export.py | tradelab v1.3 | f5a8bad758c5f30c | COMMAND: every pick with outcomes, day by day, the market around it, research, cards, dictionary; reconcile safety stop |
| tl_journal.py | tl_journal v1.0 (add-on) | aa245e22c24dfd98 | COMMAND: one readable Excel file for an export, TRADE_JOURNAL.xlsx in the export folder - Trades (newest first, in bands: result, day-by-day closes, the stock's state, the market and breadth at the pick, the hold, the story), Market days, What worked (tradelab's breakdown with its intervals), Read me; reads the export read-only (trades.parquet checked against outputs.sha256); --selftest |
| tl_portfolio.py | tradelab v1.2 | b15f0570f3aca067 | the money layer: sizing rules, capital, cash, integer shares, limits, equity, drawdown (v1.1: unfunded trades are skipped, not open) |
| tl_research.py | tradelab v1.3 | d06e2b3809d13f58 | COMMAND + library: winners / losers / severe losers, warning signs W1-W8, purged walk-forward with Bonferroni, money-view returns and drawdowns (research-declaration-2) |
| tl_rules.py | tradelab v1.1 | 0ff123752954d1f6 | the rule language: safe expression compiler, values, functions, events, plugins; post_* refused |
| tl_run.py | tradelab v1.3.1 | 441b3fdf5fba6610 | one run's data and simulation, symbol by symbol (memory stays flat); market context per run |
| tl_sim.py | tradelab v1.3.1 | 28f532b3512cbf95 | COMMAND: replay trades bar by bar: story (the market around the trade; v1.3.1: the stock's state at the pick and every session), timeline, chart (with index panel) |
| tl_state.py | tradelab v1.3.1 | 36cfecb1f9c0c35a | COMMAND + library: the stock-state engine (primary state, events, risk flags, volatility, confidence, transitions; point in time); v1.1: what ends a state worded as the rules apply it |
| tl_watch.py | tradelab v1.3.1 | 48329f6db7f9655f | COMMAND: tonight's watchlist page - market context, stock states, pick_meta's numbers, Soul cards (reads production read-only); v1.1: --all-missing catch-up, watch_all.csv |

Read-only production dependencies (on your PC, unchanged): bigmove_deploy\open_close\stage_e_execution.py (v1.2, 158883f0af70728d - checked at start; --stage-e-any to accept another), EXECUTION_DECLARATION.json, intraday_cache.py (+ its declaration), research_common.py; bigmove_deploy\data_quality.py, panel_build.py (optional); %CACHE_DAILY_ROOT% (daily bars incl. NIFTY50 / NIFTYMIDCAP150 / NIFTYSMLCAP250 / INDIAVIX, intraday_3m, panel_oc, research2\pick_meta\PM_20261017_001, sectors\amfi\ for AMFI's lists; v1.3 watch page: paper\ledger.jsonl, paper\watchlists\WL_YYYYMMDD.md, research2\pick_meta\forward\PM_cards_YYYYMMDD.csv, optional sectors\sector_map.csv).

Python packages beyond production's: pypdf, openpyxl (AMFI lists); pdfplumber, reportlab (the tests).
