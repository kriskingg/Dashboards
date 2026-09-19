# D03 — ETF / Kotak Private Operator Dashboard

## What this is

D03 is the private live ETF/Kotak operator dashboard on `lakshmidevi`.

Unlike D01/D02, this is not merely a paper-research presentation surface. It reads live broker/account state and contains guarded live operator workflows. Its backend is therefore tightly coupled to ETF broker/order safety.

## URL and server

| Item | Value |
|---|---|
| Dashboard ID | D03 |
| URL | `https://etf-trader.tailabfd53.ts.net/` |
| Server | `lakshmidevi` |
| OCI public IP | `141.148.219.153` |
| Tailscale node | `etf-trader` |
| Local origin | `127.0.0.1:8080` |
| External ingress | Tailscale Serve, tailnet only |
| Cloudflare on this host | none observed |

## End-to-end runtime path

```text
Approved personal device
  -> Tailscale tailnet
  -> https://etf-trader.tailabfd53.ts.net/
  -> Tailscale Serve
  -> 127.0.0.1:8080
  -> Uvicorn
  -> src.dashboard.app_pro:app
  -> D03 root operator UI
```

## Source ownership

| Item | Value |
|---|---|
| Repository | `kriskingg/etf-invest-engine` |
| Intended production authority | `main` |
| Current GitHub `main` HEAD re-verified 2026-09-19 | `e52aa2ab7f8d4a260027778982b6abfe69203225` |
| Current GitHub feature-branch HEAD re-verified 2026-09-19 | `b2def17c7746656ede1a9514551fe276a21e7c40` |
| Runtime checkout | `/home/ubuntu/kotak` |
| Observed deployed branch on 2026-09-19 | `platform-v2-v03-preimplementation-20260916` |
| Observed deployed HEAD | `306bd5c20dc48c682dab8c85ce1b2c677f97f66a` |
| Application | `src.dashboard.app_pro:app` |

## Critical branch/deployment warning

The GitHub feature branch has advanced beyond the last observed deployed HEAD below. Treat the deployed `306bd5c...` value as timestamped server evidence, not the current GitHub branch tip. Re-audit the host before any deployment-sensitive conclusion.

The live checkout was observed on a feature/preimplementation branch while the weekday 08:57 IST deployment cron performs:

```text
git fetch origin
git reset --hard origin/main
```

Before changing D03, reconcile what code is intended to be production-authoritative. Do not let the cron silently choose architecture.

## If you want to change this dashboard

### Composition / which features are active

Edit first:

```text
src/dashboard/app_pro.py
```

This is the composition root.

### Root dashboard backend, authentication, broker state, order gates

Edit:

```text
src/dashboard/app_hardened.py
```

### Main root HTML template

Edit:

```text
src/dashboard/templates/index.html
```

### Professional UI

Edit:

```text
src/dashboard/professional_ui.py
```

### Runtime browser fixes

Edit:

```text
src/dashboard/ui_runtime_fix.py
```

### Sell/order safety

Edit carefully:

```text
src/dashboard/safety.py
```

### Broker position/quote reconciliation

Edit:

```text
src/dashboard/position_reconciliation.py
src/dashboard/position_reconciliation_ui.py
```

### Derivatives

Use the derivatives router/template/safety modules under `src/dashboard/`.

### Basket workflows

Use:

```text
src/dashboard/basket.py
src/dashboard/basket_ui.py
src/dashboard/basket_resolver.py
src/dashboard/basket_safety.py
```

### ETF/futures conversion

Use the currently wired conversion-v2 modules. Legacy conversion API remains for compatibility, so inspect `app_pro.py` before changing conversion behavior.

### Silver operator/reference functions

Use:

```text
src/dashboard/silver_automation.py
src/dashboard/silver_automation_ui.py
```

### Startup / ingress

Use:

```text
scripts/start_dashboard.sh
scripts/status_dashboard.sh
scripts/stop_dashboard.sh
scripts/restart_dashboard.sh
```

## Required tests

Relevant dashboard/safety tests include:

```text
test_dashboard_hardened_app.py
test_dashboard_safety.py
test_order_safety.py
test_dashboard_derivative_safety.py
test_dashboard_professional_ui.py
test_dashboard_ui_runtime_fix.py
test_position_reconciliation.py
test_position_reconciliation_ui.py
test_basket_api.py
test_basket_resolver.py
test_basket_safety.py
test_basket_ui.py
test_conversion_v2.py
test_dashboard_conversion*.py
test_silver_automation_dashboard.py
```

Any backend/order-path change requires the full related safety suite. Do not use a live broker order as a deployment test.

## Data dependencies

Core production authority includes:

```text
AWS DynamoDB -> ETF strategy/campaign state
Kotak        -> actual broker positions/orders/holdings
OCI Vault    -> secrets
GitHub       -> source
Tailscale    -> private ingress/control-plane identity
```

Host-local state also exists:

```text
/home/ubuntu/kotak/data/session_store.json
/home/ubuntu/kotak/data/dashboard_config.json
/home/ubuntu/kotak/data/platform_v2_operational.db
/home/ubuntu/kotak/data/logs/
/home/ubuntu/kotak/data/research/
```

## Reboot dependency

Tailscale Serve is persistent, but the current Uvicorn process was traced to an SSH session. No dashboard-specific reboot-persistent service/hook was found during the audit.

Therefore:

```text
Tailscale can be healthy after reboot
while D03 origin 127.0.0.1:8080 is down.
```

This must be fixed deliberately, not with a public fallback.

## Repository decision

**Keep D03 in `etf-invest-engine`.**

Its backend is directly coupled to live broker truth, order safety, reconciliation and ETF production semantics. Moving the backend would create a new cross-repository safety boundary.

A presentation-only split may be considered later only after a stable internal API owns all live broker/order safety.

## Before changing D03

- [ ] Resolve deployed feature branch vs intended `main`.
- [ ] Check whether the next weekday cron reset is safe.
- [ ] Identify presentation vs broker/safety change.
- [ ] Change authoritative source, not deployed generated artifacts.
- [ ] Run dashboard + safety tests.
- [ ] Do not test by placing a real order.
- [ ] Verify local `127.0.0.1:8080`.
- [ ] Verify Tailscale route.
- [ ] Record new deployed SHA.
- [ ] Update Dashboards registry.

## Deep evidence

See:

- [../D03-etf-kotak-private-dashboard.md](../D03-etf-kotak-private-dashboard.md)
- [../LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md](../LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md)
- [../DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md](../DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md)
