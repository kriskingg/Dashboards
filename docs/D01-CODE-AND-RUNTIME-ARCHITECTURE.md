# D01 — Complete Code and Runtime Architecture

**Dashboard:** NIFTY Paper Research  
**URL:** `https://triumph-events-chair-problems.trycloudflare.com/live_latest.html`  
**Ownership status:** PROVEN  
**Architecture review date:** 2026-09-18  
**Source repository:** `kriskingg/etf-invest-engine`  
**Active deployment branch:** `chatgpt/statistical-options-buying-basket-v1`  
**Verified deployment HEAD:** `97f51a865a71bd44ddadd2007f609b4c083556a0`

This document is the detailed code map for D01. It complements the shorter ownership record in
[D01-nifty-paper-research.md](D01-nifty-paper-research.md).

The source code remains the executable truth. This document records where every major responsibility lives,
how data moves through the system, which files are runtime copies versus Git-tracked source, and which
services/timers own each stage.

---

## 1. System purpose and hard boundary

The NIFTY multi-shadow system is a **paper-research / forward-testing system** over recorded Kotak market data.

The module README explicitly states that it does **not** call broker order, login, portfolio, modify, cancel,
or transaction endpoints.

Its normal flow is:

```text
Kotak token-only REST recorder
        |
        v
5-second NIFTY spot + option observations
        |
        v
SQLite research-YYYY-MM-DD.sqlite3
        |
        v
NIFTY multi-shadow live paper engine
        |
        +--> signals
        +--> paper trades
        +--> ratio-hedge challenger state
        +--> live dashboard state
        |
        v
live_latest.html
        |
        v
shared HTTP/control server
        |
        v
Cloudflare quick tunnel
        |
        v
Browser
```

The engine is paper-only. The browser dashboard is a reporting/monitoring surface, not the execution owner.

---

## 2. Canonical source versus deployed runtime copy

### Canonical Git-tracked source

```text
kriskingg/etf-invest-engine
branch: chatgpt/statistical-options-buying-basket-v1

nifty-options-paper-research/
  analysis/
    nifty_multi_shadow/
```

### Deployed runtime copy

```text
/opt/chartink-paper/nifty-options-v2/
  analysis/
    nifty_multi_shadow/
```

At verification time the top-level deployed `analysis/` tree was untracked in the server checkout.

The critical runtime files matched the Git-tracked source byte-for-byte:

```text
dashboard.py
runtime: a6757e3087e643b0ac75f41ce993e371b3aac35b
GitHub:  a6757e3087e643b0ac75f41ce993e371b3aac35b

live.py
runtime: 4a4db2288563374e4fed4e9d8e6d6a3cb7ed138a
GitHub:  4a4db2288563374e4fed4e9d8e6d6a3cb7ed138a
```

This proves equivalence at the verification point, but future deployments must re-check hashes.

---

## 3. Complete module inventory

All paths below are relative to:

```text
nifty-options-paper-research/analysis/nifty_multi_shadow/
```

| Module | Git blob at review | Responsibility |
|---|---|---|
| `README.md` | `78dddddced63c1dfeed72111fd3a99c033bf32e4` | Human specification of forward-paper sequence, strategy rules, portfolio rules, OCI paths and research boundaries. |
| `__init__.py` | `7ff2fa565e62177c49c17774d9093d6c5a06a50b` | Package exports: engine version, config, active/derived/legacy strategy sets, priorities and `run_day`. |
| `cli.py` | `a16ca7b028a1fb3152a7148a65931d8bb92d6ddd` | Deterministic after-market replay/report CLI. Runs quality checks, replay, regime-isolated ledgers, daily and rolling reports. |
| `dashboard.py` | `a6757e3087e643b0ac75f41ce993e371b3aac35b` | Main NIFTY browser UI renderer. Produces the self-contained HTML/JS dashboard, metrics, filters, charts, strategy summaries and embedded `DASHBOARD_DATA`. |
| `dashboard_refresh.py` | `57c68ae195531dc28687e2b77179f745f73a27ca` | Replaces unconditional 10-second reload with market-hours-aware IST refresh logic. |
| `engine.py` | `4f9f066b8a4b5077ce32352710b622a5e8a76ce7` | Core deterministic research engine: data loading, aggregation, strategy definitions, signal generation, portfolio selection, quote validation, costs, entries/exits and replay. |
| `evidence_exporter.py` | `b1f10f13fd74289486cbf77067479d8386880298` | Read-only evidence exporter for forward/historical research with immutability/integrity checks, causality checks, secret-key screening and Git evidence publishing. |
| `health_monitor.py` | `bc593cfbab819dacf3ebcb72311727242197b0dd` | Fail-closed runtime audit for recorder freshness, engine freshness, database existence, ratio state, uniqueness, trade completeness, quote freshness, disk space and fingerprints. |
| `kotak_neo_api_costs.py` | `fd74390cf7cc7b615daa0a6c6e274d6334d1c976` | Accounting policy for modeled Kotak Neo API option round-trip charges. |
| `live.py` | `4a4db2288563374e4fed4e9d8e6d6a3cb7ed138a` | Live paper loop: reads recorder DB, evaluates completed-minute signals, applies entry/exit rules, maintains JSON state, merges challenger state and writes `live_latest.html`. |
| `otm_models.py` | `3136a313eeca093ad43db455ac7933784c025b8d` | Dataclasses for OTM contracts, bars and detector outputs. |
| `otm_patterns.py` | `cb6963f9bd58b3a649a69f292c96129b82fcf3f8` | Equation-based OTM chart-pattern detection on completed option bars. |
| `otm_strategy_spec.py` | `1152fbb9c06882f275d3cf4ac33747f738339df2` | OTM decision wrapper combining universe eligibility, completed-bar guard and pattern/volume detectors. |
| `otm_universe.py` | `6690453d809611cb6d9055f004b05e5ab0a741fb` | Strictly-OTM contract eligibility, executable ask < ₹10 research gate, expiry/DTE and same-day cutoff checks. |
| `otm_volume.py` | `8d6680d8cfe60ed6fe270f22f48d4d6cf581f906` | OTM volume-expansion detector using median/MAD, robust z-score, breakout, close-location and candle-quality rules. |
| `overall_strategy_numbers.py` | `3fed6c8b936f61fd5ebb26cb5057cf4b13499877` | Comprehensive historical/replay numerical extraction and repricing under the configured API-cost model. |
| `quality.py` | `8ea75af9cc69a939bc523060f7ccbb7c29ef939b` | Session data-quality gate: SQLite integrity, recorder classification, spot/option coverage, expiry count, volume presence and executable one-lot depth. |
| `ratio_hedge.py` | `5813663ddf577ca648106c5715b19dbb3f621c24` | Independent `VOLUME_2X1_HEDGE` challenger engine, separate state/fingerprint, multi-leg lifecycle and merged display/review state. |
| `replay_driver.py` | `b61d9b63299c1913c5eebabd4b924f726de1e049` | Multi-session deterministic replay and performance/risk metrics such as P&L, profit factor, drawdown and peak concurrent risk. |
| `reporting.py` | `92d191cc67d63d25eef05455f49d77a32a1d2586` | Daily/rolling report summaries, diagnostics, strategy-level performance tables and HTML report generation. |
| `research_review.py` | `69617ed2c5c96fab7a93f97b06972705bdc871dd` | Cross-session research governance review. Loads current-fingerprint live history, merges ratio state, summarizes strategies and applies evidence/review thresholds. |
| `statistical_basket.py` | `18ba983ab9d92b53132f523cd82bfcbd95741a6b` | Five long-option statistical strategies using VIX, order-flow pressure, return persistence, mean reversion and spot-vs-expected-move features. |
| `oci/` | directory | Deployment scripts, runtime wrappers, systemd units and timers. Detailed below. |

---

## 4. Core engine architecture

### Engine identity

At review time:

```text
ENGINE_VERSION = nifty-multi-shadow-3.5.0
```

### Core configuration objects

`engine.py` defines immutable dataclass configuration for:

```text
CostConfig
RiskConfig
BaselineConfig
VolumeConfig
CandleConfig
PatternConfig
StatisticalPressureConfig
EngineConfig
```

`EngineConfig` combines all of them plus `StatisticalBasketConfig`.

The engine computes a SHA-256-derived configuration fingerprint. This fingerprint is used to stop
different strategy/execution definitions from being silently mixed in one research ledger.

### Primary active independent strategies

```text
ATM_LOW_BASELINE
ATM_VOLUME_EXPANSION
SPOT_CANDLE_CONFIRMATION
CHART_PATTERN
STAT_OPTION_PRESSURE
STAT_PRESSURE_VOLUME
OTM_VOLUME_EXPANSION_LT10
OTM_CHART_PATTERN_LT10
STAT_VIX_SHOCK
STAT_ORDER_FLOW_IMBALANCE
STAT_RETURN_PERSISTENCE
STAT_MEAN_REVERSION
STAT_EXPECTED_MOVE_BREAKOUT
```

### Derived challenger strategy

```text
VOLUME_2X1_HEDGE
```

### Legacy-history-only strategy

```text
SPOT_CHART_PATTERN
```

The code deliberately distinguishes active, derived challenger and legacy-history-only strategy sets.

---

## 5. Market-data input

The engine reads the recorder SQLite database read-only.

Normal runtime input:

```text
/var/lib/chartink-paper/nifty-options-v2/data/research-YYYY-MM-DD.sqlite3
```

Main SQLite tables read by the engine:

```text
reference_bars
option_bars
```

The core loader selects 5-second NIFTY spot/reference observations and NIFTY option observations.

Option data includes, among other fields:

```text
token
symbol
option_type
strike
expiry
OHLC
volume_delta
lot_size
best bid/ask
best bid/ask quantities
executable bid/ask
cumulative executable quantities
response timestamp
REST latency
quote age
```

Optional India VIX 1-minute rows are loaded separately from `reference_bars` by
`statistical_basket.load_vix_minutes`.

Missing VIX does not stop the other statistical strategies; it disables only the VIX-dependent strategy.

---

## 6. Bar construction and no-lookahead model

The engine converts 5-second samples into completed minute bars.

Key aggregation paths:

```text
5-second NIFTY spot
    -> 1-minute spot bars
    -> 3-minute spot bars where required

5-second option observations
    -> 1-minute option bars
    -> 5-minute option bars for OTM research where required
```

Bars are end-labelled. Incomplete multi-minute bars are rejected.

The system's stated forward-research model is no-lookahead:

- signals evaluate completed bars;
- entry occurs only after a configured decision latency;
- quote/execution quality must pass;
- suppressed executable signals may retain labelled counterfactual outcomes but are not portfolio P&L.

---

## 7. Primary risk and execution-model rules

Default `RiskConfig` at review time includes:

```text
hard stop                  12%
target                     24%
trailing arm profit        12%
minimum trail lock          2%
trail distance              6%
trail tighten threshold    18%
tight trail distance        4%

first normal entry          09:30 IST
last entry                  14:45 IST
force exit                  15:10 IST
force-exit grace            30 seconds
max force-exit quote lag    10 seconds

same-day expiry cutoff      12:00 IST
maximum DTE                  8 days

decision latency             5 seconds
entry wait                   15 seconds
max spread                   2%
max quote age                3 seconds
max response latency         2500 ms
minimum 5s samples/minute    8
```

The statistical-basket strategies have their own tighter signal-time controls, including:

```text
first statistical signal    09:45 IST
last statistical signal     14:15 IST
same-day expiry cutoff      11:30 IST
minimum time to force exit   55 minutes
```

These values are code configuration, not broker orders.

---

## 8. Live-paper state machine

`live.py` is the runtime orchestrator.

### New-session state

A new state contains:

```text
engine_version
config_fingerprint
session_date
mode = LIVE_PAPER_ONLY
signals
trades
last_refresh_at
```

### Signal lifecycle

Typical statuses:

```text
PENDING_ENTRY
ACCEPTED
SUPPRESSED
REJECTED
```

Examples of suppression/rejection reasons in the current implementation include:

```text
DAILY_TRADE_CAP
DUPLICATE_STRATEGY_TIMESTAMP
ATM_CHART_PATTERN_POSITION_ALREADY_OPEN
STAT_STRATEGY_POSITION_ALREADY_OPEN
STRATEGY_COOLDOWN
NO_ELIGIBLE_CONTRACT
NO_ENTRY_QUOTE
```

The forward-research default currently has no global daily trade cap and no same-strategy cooldown,
but the code retains the generic gates.

### Trade lifecycle

Main statuses:

```text
OPEN
CLOSED
OPEN_UNRESOLVED
```

Exit reasons can include:

```text
TARGET
STOP
TRAILING_STOP
FORCED_EXIT
NO_EXECUTABLE_FORCE_EXIT
```

The system will not invent a 15:10 fill. If a sufficiently fresh executable quote is unavailable after
the grace window, the paper trade becomes `OPEN_UNRESOLVED`.

---

## 9. Entry and contract selection

For a pending signal, the live engine:

```text
signal decision
  -> wait decision_latency_seconds
  -> search within entry_wait_seconds
  -> select eligible expiry/strike/side
  -> validate quote quality
  -> require executable quote
  -> use ask for long-option paper entry
  -> use actual lot_size from recorded row
  -> calculate hard stop and target
  -> create OPEN paper trade
```

Same-day expiry is allowed only before the relevant strategy cutoff. Later signals require a later expiry.

OTM strategies add strict gates including:

```text
strictly OTM
0 < executable ask < ₹10
DTE <= 8
same-day contract disallowed after noon
completed bars only
```

---

## 10. Exit and mark-to-market model

Open trades are walked through recorded quotes chronologically.

For each valid quote the engine updates:

```text
last_quote_time
peak_bid
current_bid
unrealized_net_pnl
current_net_r
trailing_armed
effective_stop
```

For long options:

- entry uses executable ask;
- exit/mark uses executable bid;
- modeled costs are deducted;
- `Net R` is net P&L divided by original rupees at risk.

Target is checked before trailing/stop logic on each processed quote.

---

## 11. Cost model

`CostConfig` and `kotak_neo_api_costs.py` define modeled option costs.

At review time the code model includes:

```text
brokerage_per_order = 0
exchange + IPFT rate
SEBI turnover fee
GST
sell-side STT
buy-side stamp duty
tick size = ₹0.05
```

The dashboard and replay results are research results after these modeled costs.

Any future tariff change must be treated as a strategy/accounting-regime change and revalidated.

---

## 12. Ratio-hedge challenger architecture

`ratio_hedge.py` is intentionally separate from the main state.

Identity:

```text
strategy: VOLUME_2X1_HEDGE
engine version: volume-2x1-hedge-1.1.3
mode: LIVE_PAPER_ONLY_CHALLENGER
source strategy: ATM_VOLUME_EXPANSION
```

Default leg ratio:

```text
2 directional-side lots
1 opposite-side hedge lot
```

It has its own:

- configuration;
- fingerprint;
- state file;
- signal ledger;
- trade ledger;
- exit/lifecycle calculations.

State path:

```text
/var/lib/chartink-paper/nifty-multi-shadow/
  ratio_hedge/live/YYYY-MM-DD/state.json
```

`merge_ratio_state` creates a display/review copy that combines base state with challenger trades/signals.
The independent underlying ledgers remain separate.

---

## 13. Statistical-basket architecture

`statistical_basket.py` owns five long-option research strategies:

```text
STAT_VIX_SHOCK
STAT_ORDER_FLOW_IMBALANCE
STAT_RETURN_PERSISTENCE
STAT_MEAN_REVERSION
STAT_EXPECTED_MOVE_BREAKOUT
```

Its frozen configuration includes:

- restricted signal window;
- same-day expiry cutoff;
- minimum remaining time before force exit;
- top-of-book freshness/spread/latency gates;
- robust lookback;
- regime window;
- variance-ratio horizon;
- VIX shock thresholds;
- order-flow thresholds;
- persistence/reversion thresholds;
- expected-move efficiency thresholds.

These are generated as research signals and then pass through the live-paper execution model.

---

## 14. OTM research architecture

OTM research is split into small modules deliberately:

```text
otm_models.py
    data structures

otm_universe.py
    contract eligibility

otm_volume.py
    volume expansion detector

otm_patterns.py
    chart-pattern detector

otm_strategy_spec.py
    combines eligibility + completed-bar guard + detector
```

This separation lets the universe gate and signal detector be tested independently.

---

## 15. Session data-quality gate

`quality.py` determines whether a captured session is usable for forward research.

Checks include:

```text
SQLite integrity == ok
capture completed
session date matches
NIFTY spot 5-second coverage >= 98%
option 5-second coverage >= 95%
at least two expiries captured
incremental volume exists
all recorded depth can fill one lot
```

The option-coverage check currently expects at least 18 option rows in a 5-second bucket.

If the daily quality gate fails, replayed portfolio outcomes are suppressed from valid research use rather than silently treated as clean evidence.

---

## 16. After-market replay and report architecture

`cli.py` owns deterministic after-market replay.

Flow:

```text
research SQLite + recorder health
        |
        v
build_quality_report()
        |
        v
engine.run_day()
        |
        +--> signals.csv
        +--> outcomes.csv
        +--> data_quality.json
        +--> metadata.json
        |
        v
regime-isolated cumulative ledgers
        |
        +--> daily report
        +--> daily_latest.html
        +--> rolling 5-session weekly_latest.html
```

Configuration-regime ledgers are stored under:

```text
/var/lib/chartink-paper/nifty-multi-shadow/
  ledger/regimes/<config-fingerprint>/
```

Data-quality history remains global because it describes the market capture, while signal/outcome history is regime-specific.

---

## 17. Research review/governance

`research_review.py` reads live history for the current configuration fingerprint and produces
cross-session research summaries.

It does not automatically promote a strategy.

The code explicitly says:

```text
Never auto-promote. Freeze a new fingerprint and forward-test as a paper challenger.
```

Review states include evidence-collection/development/review categories based on sample size,
mean/median R, profit factor, positive weeks and recovery behavior.

---

## 18. Evidence exporter

`evidence_exporter.py` is a read-only post-market evidence pipeline.

Its stated responsibilities include:

- compact forward/historical evidence export;
- cryptographic integrity;
- observation-time causality;
- exact CE/PE batching;
- OTM cutoff evidence;
- ratio-hedge leg lifecycle evidence;
- Git evidence publishing;
- forbidden-secret-key filtering.

It searches multiple locations to resolve the deployed Git SHA, including the runtime Git checkout and
deployment SHA marker files.

It must remain read-only with respect to broker/trading behavior.

---

## 19. Runtime health monitor

`health_monitor.py` audits:

```text
session date
recorder failures
database existence
recorder freshness
paper-engine freshness
ratio-hedge freshness
ratio state presence/fingerprint
unique trade IDs
unique signal IDs
closed-trade completeness
open-trade quote freshness
free disk
main config fingerprint
```

During the monitoring-critical market window it treats stale recorder/engine state as critical.

Output:

```text
/var/lib/chartink-paper/nifty-multi-shadow/
  monitor/health_latest.json
```

Exit contract:

```text
0 = healthy
1 = warning
2 = critical
```

---

## 20. Dashboard rendering architecture

`dashboard.py` renders the browser page.

Important characteristics:

- self-contained HTML;
- embedded `DASHBOARD_DATA`;
- strategy/trade/signal metrics;
- filterable trade/signal views;
- JavaScript charts and summaries;
- historical retained-trade views;
- strategy explanations;
- NIFTY/MCX navigation;
- paper-only indicator;
- recorder/freshness status.

The distinct heading:

```text
NIFTY paper research
```

is produced here.

`dashboard_refresh.py` patches the dashboard's legacy unconditional reload so that:

- during the configured weekday market window it refreshes every 10 seconds;
- outside the window it pauses reloads;
- it periodically rechecks whether the market refresh window has reopened.

---

## 21. Generated runtime state and report tree

Primary state root:

```text
/var/lib/chartink-paper/nifty-multi-shadow
```

Important logical paths:

```text
live/YYYY-MM-DD/state.json
ratio_hedge/live/YYYY-MM-DD/state.json
daily/YYYY-MM-DD/
ledger/
ledger/regimes/<config-fingerprint>/
monitor/health_latest.json
reports/live_latest.html
reports/daily_latest.html
reports/weekly_latest.html
```

The generated HTML files are artifacts, not authoritative source.

---

## 22. OCI runtime wrappers and deployment inventory

The source `oci/` directory contains:

| File | Blob | Role |
|---|---|---|
| `deploy_server.sh` | `6fe661535d982b0a9321dbb487f05853e22fee8d` | Server deployment/install helper. |
| `migrate_uncapped_state.py` | `aae4af0d0ca8b7069a59184cc817446b36afaa83` | State migration helper for the uncapped forward-research model. |
| `nifty-multi-shadow-dashboard.service` | `52d759d4c95a9135b7a35227de3bff288e5fb9c6` | Original/local dashboard server unit definition in source. |
| `nifty-multi-shadow-evidence-exporter.service` | `ace7deaa807b7e82db1e68b4c20a77e7e15ca047` | Evidence exporter service. |
| `nifty-multi-shadow-evidence-exporter.timer` | `be1479c678ed464046baf429250b8a3f25acaa0a` | Evidence exporter schedule. |
| `nifty-multi-shadow-live.service` | `8e6a4f9d8aa973f369ebeb1901823b4783792cec` | Live-paper engine service definition. |
| `nifty-multi-shadow-live.timer` | `2ae9b2661c97ccb6019d4f0d1644417f0f87e71f` | Starts live-paper engine before its signal window. |
| `nifty-multi-shadow.service` | `dad4bc65640977f145296cdb0b6e1a22e14ff9a6` | After-market replay/report service. |
| `nifty-multi-shadow.timer` | `dd9fd8fa3e23d0ebc4d2f63e33aa0e0c63f43179` | After-market replay/report schedule. |
| `nifty-paper-health-monitor.service` | `58617b6e5afd215b352c584cd91f501c30069f6f` | Health-monitor service. |
| `nifty-paper-health-monitor.timer` | `a1edc719969cb20927314ee403642285ec3361fe` | Health-monitor schedule. |
| `nifty-research-review.service` | `2f10d63e2f7c61ff35c66c8c00931b6b885fdd50` | Research-review service. |
| `nifty-research-review.timer` | `0c20540b9b2b6dbc7c4289f88c75aba994ec6a25` | Research-review schedule. |
| `run_daily.sh` | `21a16153fcd26a29e359e4f76ae90a502d6de46e` | Daily deterministic replay wrapper. |
| `run_evidence_exporter.sh` | `30688be566d2d12a1c4005de0f7191c42ac19d3f` | Evidence-export wrapper. |
| `run_health_monitor.sh` | `acce525fece22ca27017fabdd36236f09c565242` | Health-monitor wrapper. |
| `run_live.sh` | `728a557a20bceb24420e36599b0d131f0369c196` | 10-second live-paper loop and final settlement pass. |
| `run_review.sh` | `3684bd2903e2e5f57d2b47b5c430730a05b3ed6e` | Research-review wrapper. |

---

## 23. Live service execution sequence

Current live wrapper:

```text
run_live.sh
```

uses:

```text
APP=/opt/chartink-paper/nifty-options-v2
SOURCE_STATE=/var/lib/chartink-paper/nifty-options-v2
STATE=/var/lib/chartink-paper/nifty-multi-shadow
PYTHON=/opt/chartink-paper/options-venv/bin/python
```

It waits until 09:20 IST, then repeatedly runs the live-paper cycle approximately every 10 seconds.

For each cycle:

```text
research-YYYY-MM-DD.sqlite3
health.json
        |
        v
python -m analysis.nifty_multi_shadow.live
        --database ...
        --recorder-health ...
        --state-dir /var/lib/chartink-paper/nifty-multi-shadow
```

It performs one final fresh-data settlement pass at/after 15:10:30 IST and then exits.

---

## 24. Current systemd live service

Verified running definition:

```text
nifty-multi-shadow-live.service

User=chartink-paper
Group=chartink-paper
WorkingDirectory=/opt/chartink-paper/nifty-options-v2
Environment=PYTHONUNBUFFERED=1
Environment=PYTHONDONTWRITEBYTECODE=1
ExecStart=/opt/chartink-paper/nifty-options-v2/analysis/nifty_multi_shadow/oci/run_live.sh

ReadOnlyPaths=/opt/chartink-paper/nifty-options-v2
              /var/lib/chartink-paper/nifty-options-v2
ReadWritePaths=/var/lib/chartink-paper/nifty-multi-shadow

MemoryMax=450M
TasksMax=32
CPUQuota=75%
```

---

## 25. Serving architecture is deliberately separate

The current HTTP/control service is not the NIFTY generator.

Serving path:

```text
/opt/chartink-paper/mcx-paper-research
        |
        v
mcx_paper.control_server
        |
        v
127.0.0.1:8765
        |
        v
/var/lib/chartink-paper/nifty-multi-shadow/reports
        |
        +--> live_latest.html
        +--> mcx_latest.html
```

The NIFTY page is generated by D01 source but served by this shared server.

This is a critical architectural fact.

---

## 26. Ingress architecture

At verification time Cloudflare quick tunnel runs against:

```text
http://127.0.0.1:8765
```

and exposes:

```text
https://triumph-events-chair-problems.trycloudflare.com
```

Therefore:

```text
/live_latest.html -> NIFTY generated artifact
/mcx_latest.html  -> separate MCX generated artifact
```

A common hostname does not mean common generator ownership.

---

## 27. Repository/branch identity

Server runtime checkout:

```text
/opt/chartink-paper/nifty-options-v2
```

Git remote:

```text
git@github.com:kriskingg/etf-invest-engine.git
```

Active branch:

```text
chatgpt/statistical-options-buying-basket-v1
```

Verified HEAD:

```text
97f51a865a71bd44ddadd2007f609b4c083556a0
```

Historical branches such as `codex/nifty-paper-research-backup` are lineage/reference only and are not
the active deployment owner.

---

## 28. Dirty-worktree warning

At review time the deployed Git checkout had extensive local differences:

- modified documentation;
- many deleted documentation files;
- modified deployment scripts;
- untracked top-level `analysis/`;
- untracked research/audit documents;
- untracked replay/runtime material;
- `.git_sha`.

Therefore the current server is not a simple clean clone/deploy model.

Do not run destructive Git operations without a dedicated reconciliation plan.

Unsafe examples:

```text
git reset --hard
git clean -fd
git checkout .
blind git pull
blind branch switch
```

---

## 29. Safe change workflow for D01

Any permanent dashboard/engine change should follow this ownership path:

```text
1. Confirm D01 URL and current server.
2. Re-check branch, HEAD and git status.
3. Modify canonical tracked source:
   nifty-options-paper-research/analysis/nifty_multi_shadow/
4. Run focused tests.
5. Review diff and safety boundary.
6. Commit/push on intended branch.
7. Synchronize canonical source deliberately into deployed runtime analysis/.
8. Compare runtime and Git blob hashes.
9. Restart only the affected NIFTY service if required.
10. Confirm state generation.
11. Confirm local page on 127.0.0.1:8765/live_latest.html.
12. Confirm external Cloudflare page.
13. Update this ownership registry if architecture changed.
```

Do not patch generated `live_latest.html` as a permanent fix.

---

## 30. What to check when the page looks wrong

Use this order:

```text
A. Is live_latest.html freshly generated?
B. Does the generated file contain the expected UI string?
C. Does localhost:8765 serve the expected file?
D. Does the external tunnel serve the same content?
E. Does runtime dashboard.py hash match Git source?
F. Is nifty-multi-shadow-live.service healthy?
G. Is recorder health fresh?
H. Is the research SQLite present and growing during market time?
```

This isolates generator, filesystem, serving and ingress failures instead of mixing them.

---

## 31. Source-of-truth hierarchy

For D01:

```text
Strategy/code truth:
GitHub canonical source on active branch

Deployment truth:
actual server branch/HEAD + runtime file hashes + systemd

Research state truth:
files under /var/lib/chartink-paper/nifty-multi-shadow

Generated UI truth:
live_latest.html generated from runtime state

Serving truth:
localhost service response

External-access truth:
Cloudflare/Tailscale ingress response
```

Never infer one layer purely from another.

---

## 32. Final ownership statement

```text
D01 = NIFTY Paper Research

Generator code owner:
kriskingg/etf-invest-engine
chatgpt/statistical-options-buying-basket-v1
nifty-options-paper-research/analysis/nifty_multi_shadow/

Runtime:
chartink-paper
/opt/chartink-paper/nifty-options-v2/analysis/nifty_multi_shadow/

State:
/var/lib/chartink-paper/nifty-multi-shadow/

Primary generated page:
/var/lib/chartink-paper/nifty-multi-shadow/reports/live_latest.html

HTTP/control serving layer:
/opt/chartink-paper/mcx-paper-research
mcx_paper.control_server
127.0.0.1:8765

External ingress at verification:
Cloudflare quick tunnel
triumph-events-chair-problems.trycloudflare.com

Mode:
PAPER RESEARCH ONLY
```

If any of these values change, update the D01 registry record immediately.
