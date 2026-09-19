# D05 — Mutual Funds Analytics / Tactical Research Dashboard

## What this is

D05 is the local Mutual Funds Analytics & Tactical Trading research platform.

It combines AMFI/NAV ingestion, technical analytics, Mutual Fund Intelligence,
Turnaround Hunter, chart-pattern research, market-direction/breadth analysis,
screeners, comparison tools and interactive charting.

## Runtime location

| Item | Value |
|---|---|
| Dashboard ID | D05 |
| Host | local Windows workstation |
| Canonical workspace | `D:\Git_repos\mf-analytics-source` |
| Frontend | React + TypeScript + Vite |
| Frontend URL from current startup script | `http://localhost:5173` |
| Backend | FastAPI |
| Backend port from current startup script | `8000` |
| Database | PostgreSQL |
| OCI dependency | none proven |
| Public URL | none proven |
| Legacy workspace | `D:\Dhan\Mutual_funds` — retired as runtime authority after 2026-09-19 migration |

## Source ownership

| Item | Value |
|---|---|
| Repository | `kriskingg/mf-analytics-source` |
| Branch | `main` |
| HEAD verified 2026-09-19 | `5366b9000d0f93af8e4e635b4fc551f0a85d0915` |
| Frontend root | `app/frontend/` |
| Backend root | `app/backend/` |
| Startup scripts | `app/scripts/` |
| Preserved local-source branch | `preserve/d05-local-source-20260919` |
| Preserved local-source commit | `478e7a018ba0d587915445d4f7decfb056c63a0d` |
| Path-cleanup work | PR #1, branch `chatgpt/d05-path-cleanup-20260919`; pending integration/testing, not yet `main` |

## 2026-09-19 local migration verification

D05 was migrated from the legacy local workspace into the canonical Git workspace:

```text
OLD runtime/workspace: D:\Dhan\Mutual_funds
NEW canonical path:    D:\Git_repos\mf-analytics-source
```

Verified after migration:

- `D:\Git_repos` is a real Windows directory, not a junction/reparse-point alias;
- repository remote remains `https://github.com/kriskingg/mf-analytics-source.git`;
- branch remains `main`;
- HEAD remains `5366b9000d0f93af8e4e635b4fc551f0a85d0915`;
- all old-source files were present in the new destination during migration verification;
- D05 tracked-file old-versus-new comparison reported **0 differences**;
- `.venv` was rebuilt so `sys.executable` and `sys.prefix` resolve under `D:\Git_repos\mf-analytics-source`;
- `http://127.0.0.1:8000/docs` returned HTTP 200;
- `http://127.0.0.1:5173` returned HTTP 200.

Temporary migration backups may exist until cleanup is explicitly approved. They are safety copies, not active code/runtime authorities.

Large local NAV/market/bhavcopy data, logs, `.venv`, `node_modules`, ZIP deliverables and generated artifacts may physically coexist inside the working tree. Do **not** use blind `git add .` / `git add -A`; audit ignore/staging scope first.

## Post-migration source preservation and Git boundary audit

After the filesystem/runtime migration, the local D05 working tree was audited before any broad staging.

Verified durable results:

- the meaningful unpublished local source consisted of 10 tracked source changes plus one new regression test and two application assets;
- those 13 files were explicitly staged, checked and committed as `478e7a018ba0d587915445d4f7decfb056c63a0d`;
- the commit was pushed to `preserve/d05-local-source-20260919`, so the pre-integration local source is now recoverable from GitHub;
- after preservation, the tracked working-tree content had no real diff from that preservation commit;
- large NAV/history, bhav/market, Parquet, database, logs, environments, dependency trees, ZIP/back-up and generated-result material remains local/non-source;
- the audit found no Git-tracked file larger than 20 MB;
- plain `.csv` is not globally ignored, so any future CSV must be classified before staging;
- local-only generated backup/kit/result directories are excluded locally and are not treated as canonical source.

The intended repository boundary is:

```text
GitHub
  -> application source
  -> tests
  -> scripts
  -> SQL/schema
  -> current documentation
  -> small required assets/config/reference material

Local only
  -> NAV/history and bhav/market datasets
  -> Parquet and live database state
  -> logs/caches
  -> virtual environments and node_modules
  -> ZIPs/backups/generated results
```

The path-cleanup work in PR #1 is intentionally still separate from the preservation commit. The next gate is to integrate both histories, resolve any overlap deliberately, run backend/frontend validation, verify old-path references are gone, and only then merge to `main`.

Deep evidence: [../D05_LOCAL_MIGRATION_VERIFICATION_2026-09-19.md](../D05_LOCAL_MIGRATION_VERIFICATION_2026-09-19.md).

Session continuation / turnover: [../D05_SESSION_TURNOVER_2026-09-19.md](../D05_SESSION_TURNOVER_2026-09-19.md).

## End-to-end local path

```text
Browser
  -> localhost:5173
  -> React/Vite frontend
  -> app/frontend/src/*
  -> FastAPI backend localhost:8000
  -> app/backend/app/*
  -> PostgreSQL + analytics tables
  -> AMFI/data/update engines
```

The current `app/scripts/start_platform.ps1` also starts/checks PostgreSQL and
spawns `backend/scripts/auto_update.py`.

## If you want to change this dashboard

### Main application/navigation

```text
app/frontend/src/App.tsx
```

### Major dashboard/workspace modules

```text
app/frontend/src/components/TurnaroundHunterView.tsx
app/frontend/src/components/ChartPatternsView.tsx
app/frontend/src/components/MarketDirectionView.tsx
app/frontend/src/components/MarketRadarView.tsx
app/frontend/src/components/MFIntelligenceView.tsx
app/frontend/src/components/AdvancedScreenerView.tsx
app/frontend/src/components/ScreenerView.tsx
app/frontend/src/components/MomentumView.tsx
app/frontend/src/components/CategoriesView.tsx
app/frontend/src/components/SignalCrossoversView.tsx
app/frontend/src/components/CompareFundsView.tsx
app/frontend/src/components/WatchlistsView.tsx
app/frontend/src/components/StrategyBuilderView.tsx
app/frontend/src/components/MutualFundChart.tsx
```

### Frontend API client

```text
app/frontend/src/services/api.ts
app/frontend/src/services/chartExtrasApi.ts
```

### Backend composition

```text
app/backend/app/main.py
```

### Backend business/API modules

Use the matching router/engine under:

```text
app/backend/app/routes/
app/backend/app/
app/backend/scripts/
```

Do not reproduce calculation logic in React when the backend is the analytical
authority.

## Tests

The repository contains a substantial backend regression suite, including:

```text
app/backend/tests/test_turnaround_hunter.py
app/backend/tests/test_market_direction.py
app/backend/tests/test_market_radar_*.py
app/backend/tests/test_mf_chart_patterns_*.py
app/backend/tests/test_mf_decision_*.py
app/backend/tests/test_mf_intelligence_*.py
app/backend/tests/test_mf_signal_crossovers_*.py
app/backend/tests/test_mf_rsi_ema_signals_*.py
app/backend/tests/test_screener_engine.py
app/backend/tests/test_fund_chart_extras_*.py
```

Frontend production changes should also pass the Vite build.

## Startup / scheduling

Current startup script:

```text
app/scripts/start_platform.ps1
```

It:

- checks/starts PostgreSQL;
- launches FastAPI on port 8000;
- launches Vite on port 5173;
- starts the updater.

Task Scheduler definitions exist in:

```text
app/scripts/install_startup_task.ps1
```

They define startup-at-logon and a daily update task, but the script defaults to
preview mode unless explicitly installed.

## Documentation discrepancy to remember

The master README describes a FastAPI backend on port `8010`, while the current
startup script launches port `8000`.

For runtime work, verify the actual local process/start script before assuming
either number. Code/runtime evidence overrides prose.

## Data dependencies

The platform depends on:

- PostgreSQL mutual-fund data;
- AMFI NAV/master ingestion;
- technical feature tables;
- signal/pattern tables;
- scheduler/update jobs;
- local frontend/browser state for UI preferences.

This dashboard is independent of `lakshmidevi` and `chartink-paper`.

## Repository decision

**Keep D05 in `mf-analytics-source`.**

The frontend, analytical backend, schema, tests and update pipeline form one
coherent domain. There is no current reason to move its UI into the generic
Dashboards registry repo.

## Before changing D05

- [ ] Confirm current `main` SHA.
- [ ] Confirm actual local backend/frontend ports.
- [ ] Classify presentation vs analytical-rule change.
- [ ] Update backend tests if analytical behavior changes.
- [ ] Run relevant backend test suite.
- [ ] Run frontend build.
- [ ] Verify API/UI locally.
- [ ] Update this registry if architecture/ports/repo boundaries change.
