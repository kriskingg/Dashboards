# D04 — ETF Platform-v2 Dashboard

## What this is

D04 is the newer ETF/Kotak Platform-v2 user interface mounted at `/v2/`.

It is a **different frontend** from D03, but it currently runs inside the same FastAPI/Uvicorn process and relies on the ETF Platform-v2 backend in the same repository.

## URL and server

| Item | Value |
|---|---|
| Dashboard ID | D04 |
| URL | `https://etf-trader.tailabfd53.ts.net/v2/` |
| Server | `lakshmidevi` |
| OCI public IP | `141.148.219.153` |
| Local origin | same `127.0.0.1:8080` as D03 |
| External ingress | same tailnet-only Tailscale Serve |
| Runtime process | same `src.dashboard.app_pro:app` Uvicorn process |

## End-to-end runtime path

```text
Approved personal device
  -> Tailscale
  -> /v2/
  -> Uvicorn src.dashboard.app_pro:app
  -> install_spa(app, WEB_DIST)
  -> src/dashboard/spa.py
  -> web/dist
  <- built from web/src
  -> /api/v2 backend
  -> src/platform_v2/*
```

D04 is not a second strategy engine.

## Source ownership

| Item | Value |
|---|---|
| Repository | `kriskingg/etf-invest-engine` |
| Runtime checkout | `/home/ubuntu/kotak` |
| Observed deployed branch | `platform-v2-v03-preimplementation-20260916` |
| Current GitHub feature-branch HEAD re-verified 2026-09-19 | `b2def17c7746656ede1a9514551fe276a21e7c40` |
| Last observed deployed HEAD | `306bd5c20dc48c682dab8c85ce1b2c677f97f66a` |
| Frontend source | `web/src/` |
| Backend source | `src/platform_v2/` |
| Integration | `src/dashboard/app_pro.py`, `src/dashboard/spa.py` |

## If you want to change this dashboard

### Navigation/routes

Edit:

```text
web/src/App.tsx
```

### Pages and presentation

Edit:

```text
web/src/pages.tsx
```

### Silver V2 workspace

Edit:

```text
web/src/components/SilverV2Workspace.tsx
web/src/components/ObservedBrokerSilver.tsx
web/src/components/StatusStrip.tsx
web/src/silverReady.ts
```

### Layout

Edit:

```text
web/src/layout/AppShell.tsx
```

### Browser API client

Edit:

```text
web/src/api.ts
```

### Browser type contracts

Edit:

```text
web/src/types.ts
```

### Styling

Edit:

```text
web/src/styles.css
```

### Trade drawer UI

Edit:

```text
web/src/components/TradeDrawer.tsx
```

The browser must not become the authority for risk/order safety.

### Backend API

Edit:

```text
src/platform_v2/api_router.py
```

plus the appropriate Platform-v2 domain/service module.

### Broker truth / normalization

Use Platform-v2 broker/query/quote modules. Do not duplicate broker normalization in React.

### Operational SQLite/state

Use:

```text
src/platform_v2/operational_store.py
```

and its service layer.

### Route mount/deep links

Use:

```text
src/dashboard/app_pro.py
src/dashboard/spa.py
```

### Never edit directly

Do not make persistent changes under:

```text
web/dist/
```

That is compiled output.

## Required tests

The Platform-v2 suite includes:

```text
test_platform_v2_api_router.py
test_platform_v2_broker_sync.py
test_platform_v2_broker_truth_ui.py
test_platform_v2_contract_registration.py
test_platform_v2_contracts.py
test_platform_v2_domain.py
test_platform_v2_exposure.py
test_platform_v2_foundation.py
test_platform_v2_instrument_search.py
test_platform_v2_integrity.py
test_platform_v2_ledger_service.py
test_platform_v2_market_data_service.py
test_platform_v2_performance_service.py
test_platform_v2_reservation_service.py
test_platform_v2_risk_workflow_scenario.py
test_platform_v2_scrip_master_status.py
test_platform_v2_silver_ready.py
test_platform_v2_silver_shadow.py
test_platform_v2_sqlite_services.py
test_platform_v2_trade_preview.py
test_platform_v2_v03_architecture.py
```

## Data dependencies

D04 consumes live broker/account data through Platform-v2 services and also uses local operational SQLite:

```text
/home/ubuntu/kotak/data/platform_v2_operational.db
```

It shares the broader ETF domain dependencies with D03:

- Kotak broker;
- OCI Vault;
- source checkout;
- Tailscale ingress;
- ETF safety/domain services.

## D03 relationship

D03 and D04 share:

- server;
- process;
- port;
- broker/domain backend;
- ingress.

They differ in presentation architecture.

```text
D03: server-rendered/injected legacy-root operator UI
D04: React/TypeScript/Vite frontend + /api/v2
```

Do not remove one merely because both expose ETF/operator information. Feature coverage and safety parity must be compared first.

## Current documentation discrepancy

The Platform-v2 `web/README.md` still describes the frontend as an isolated future shell not connected to the existing live dashboard/broker routes, while deployed branch code now mounts it into the live FastAPI application and exposes Platform-v2 API routers.

Code + tests are the authority. Treat that README language as stale until corrected.

## Repository decision

**Keep D04 backend in `etf-invest-engine` for now.**

The React frontend is a possible future extraction candidate, but only after:

1. production branch ownership is reconciled;
2. `/api/v2` becomes a stable, versioned contract;
3. D03/D04 long-term roles are decided;
4. frontend build/deploy is deterministic;
5. safety parity is proven.

If split later, separate only the presentation client unless a deliberate service architecture is approved.

## Before changing D04

- [ ] Resolve deployed branch vs production authority.
- [ ] Classify frontend vs backend/data-contract change.
- [ ] Change `web/src`, never `web/dist` directly.
- [ ] Run Platform-v2 contract/safety tests.
- [ ] Build frontend deterministically.
- [ ] Verify `/api/v2` behavior.
- [ ] Verify `/v2/` locally.
- [ ] Verify Tailscale route.
- [ ] Confirm D03 remains unaffected.
- [ ] Record source/build/deployed SHA.

## Deep evidence

See:

- [../D04-etf-dashboard-v2.md](../D04-etf-dashboard-v2.md)
- [../D03-etf-kotak-private-dashboard.md](../D03-etf-kotak-private-dashboard.md)
- [../LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md](../LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md)
- [../DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md](../DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md)
