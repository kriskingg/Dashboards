# D06 — Personal Investment Tracker Dashboard

## What this is

D06 is the local-first personal investment tracker.

It covers portfolio value, transaction entry, FIFO holdings, realised/unrealised
P&L, XIRR, asset allocation, net worth, financial-independence views, investment
plans and insights.

## Runtime location

| Item | Value |
|---|---|
| Dashboard ID | D06 |
| Host | local Windows workstation |
| Repository path documented | `D:\Git_repos\investment-tracker-app` |
| Frontend | React + TypeScript + Vite |
| Backend | FastAPI |
| Database | local SQLite |
| External exposure | localhost-only by design |
| OCI dependency | none |
| Public URL | none |

## Source ownership

| Item | Value |
|---|---|
| Repository | `kriskingg/investment-tracker-app` |
| Branch | `main` |
| HEAD verified 2026-09-19 | `8f3a24bc7f30ae10d6695a6a292447351a7db5e9` |
| Frontend root | `frontend/` |
| Backend root | `backend/` |

## Current local start contract

The current script:

```text
scripts/start-dev.ps1
```

launches:

```text
FastAPI:  http://127.0.0.1:8005
Frontend: http://127.0.0.1:5175
```

The README still mentions `5173` / `8000`. Treat the start script as the
current executable contract unless runtime evidence says otherwise.

## End-to-end path

```text
Browser
  -> 127.0.0.1:5175
  -> React/Vite
  -> frontend/src/*
  -> FastAPI 127.0.0.1:8005
  -> backend/app/*
  -> SQLite
  -> market/NAV providers where explicitly configured
```

## If you want to change this dashboard

### Main navigation

```text
frontend/src/App.tsx
frontend/src/components/Layout.tsx
```

### Dashboard home

```text
frontend/src/pages/Dashboard.tsx
```

### Other major views

```text
frontend/src/pages/Portfolio.tsx
frontend/src/pages/NetWorth.tsx
frontend/src/pages/Freedom.tsx
frontend/src/pages/Transactions.tsx
frontend/src/pages/Insights.tsx
frontend/src/pages/Plans.tsx
frontend/src/pages/Settings.tsx
```

### Transaction entry

```text
frontend/src/components/QuickAdd.tsx
```

### Browser API layer

```text
frontend/src/api.ts
```

### Backend composition

```text
backend/app/main.py
```

Backend APIs/accounting/pricing belong under `backend/app/`. The frontend must
not become the authority for accounting calculations.

## Financial truth contract

The repository explicitly defines:

- transactions are the source of financial truth;
- holdings are derived;
- FIFO is used for sell-lot matching;
- backend Python `Decimal` owns money/unit calculations;
- personal data stays local;
- V1 binds to localhost only.

UI changes must preserve those boundaries.

## Tests

Key backend tests include:

```text
backend/tests/test_fifo.py
backend/tests/test_api.py
backend/tests/test_flows.py
backend/tests/test_xirr.py
backend/tests/test_safety.py
backend/tests/test_net_worth.py
backend/tests/test_net_worth_analysis.py
backend/tests/test_financial_independence.py
backend/tests/test_market_discovery.py
backend/tests/test_kotak_pricing.py
backend/tests/test_nps_pricing.py
backend/tests/test_startup_price_refresh.py
```

There are also isolated runtime/safety scripts under `scripts/`.

## Data dependencies

D06 primarily depends on local SQLite plus explicitly approved market/NAV
providers.

It does **not** depend on either OCI dashboard server.

## Repository decision

**Keep D06 in `investment-tracker-app`.**

It is already a clean, independent domain/application repository.

## Before changing D06

- [ ] Confirm current `main` SHA.
- [ ] Use the current start script ports, not stale README ports.
- [ ] Classify UI vs accounting/data-provider change.
- [ ] Preserve transaction/FIFO/Decimal truth contracts.
- [ ] Run backend tests.
- [ ] Build/test frontend.
- [ ] Verify localhost-only binding remains intentional.
- [ ] Update this registry if runtime/deployment changes.
