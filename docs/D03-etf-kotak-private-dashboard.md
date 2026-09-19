# D03 — ETF / Kotak Private Operator Dashboard

**Status:** SOURCE + NETWORK ARCHITECTURE PROVEN; CURRENT DEPLOYED BYTES RE-VERIFICATION PENDING  
**GitHub verification:** 2026-09-19  
**Production URL:** `https://etf-trader.tailabfd53.ts.net/`

## Purpose

This is the private Kotak Neo trading/portfolio operator dashboard. It is a
different operational boundary from the NIFTY options paper-research and MCX
paper-research dashboards on `chartink-paper`.

The owning branch, `kriskingg/etf-invest-engine main`, is the **live ETF/MTF
production branch**, not merely a dashboard branch. It also owns the scheduled
ETF strategy (`pair_1`, `pair_2`, `pair_3`), campaign accounting
(`KnowYourPNL`), DynamoDB integration and production cron.

The dashboard includes portfolio/holdings/positions, broker reconciliation, guarded manual derivative workflows, ETF-to-futures conversion surfaces, basket validation, and Silver automation/reference presentation.

## Server identity

Current repository documentation identifies the host as:

```text
Public OCI IP:      141.148.219.153
Host identity:      lakshmidevi
Tailscale node:     etf-trader
MagicDNS:           etf-trader.tailabfd53.ts.net
Tailscale IPv4:     100.120.194.42   (verified 2026-09-15)
```

This is a separate OCI VM from `chartink-paper` / `80.225.234.65`.

## Network request path

```text
Approved personal device
        |
        | private Tailscale tailnet
        | HTTPS TCP 443
        v
https://etf-trader.tailabfd53.ts.net/
        |
        v
Tailscale Serve (tailnet only)
        |
        | loopback proxy
        v
http://127.0.0.1:8080
        |
        v
FastAPI/Uvicorn
src.dashboard.app_pro:app
        |
        v
Kotak dashboard application
```

Production documentation explicitly requires:

- Uvicorn loopback-only on `127.0.0.1:8080`;
- Tailscale Serve only;
- Device Approval;
- application password authentication;
- no Tailscale Funnel;
- no public Cloudflare fallback.

## Runtime/source location

Repository documentation and startup scripts use:

```text
Runtime checkout: /home/ubuntu/kotak
Repository:       kriskingg/etf-invest-engine
Branch:           main
Application:      src.dashboard.app_pro:app
```

The current GitHub `main` HEAD observed during this source review was:

```text
b1485952fd9dc5846629c59134bfda9bb9765c21
```

The operational status document records that the dashboard ingress cutover itself was validated on commit:

```text
05e48efed5fb25eedb3ce2da0a3ad055ec052288
```

Later commits extended the dashboard after that cutover. Therefore the historical cutover SHA must not be mistaken for the current deployed SHA. A fresh server check is required.

## Composition root

Primary source:

```text
src/dashboard/app_pro.py
```

The composition root imports the hardened dashboard and adds:

- derivatives router;
- conversion router;
- basket router;
- Silver automation router;
- professional UI;
- position reconciliation UI;
- conversion/basket UI;
- Silver automation UI;
- runtime UI fixes.

The root app version in current source is:

```text
1.7.1-silver-private-reference
```

## Source modules

Current primary source set includes:

```text
src/dashboard/app_pro.py
src/dashboard/app_hardened.py
src/dashboard/safety.py
src/dashboard/derivatives.py
src/dashboard/conversion.py
src/dashboard/basket.py
src/dashboard/basket_resolver.py
src/dashboard/basket_safety.py
src/dashboard/position_reconciliation.py
src/dashboard/silver_automation.py
src/dashboard/silver_automation_ui.py
src/dashboard/templates/index.html
src/dashboard/templates/derivatives.html
scripts/start_dashboard.sh
scripts/status_dashboard.sh
scripts/stop_dashboard.sh
scripts/restart_dashboard.sh
```

## Current GitHub blob identities

Observed on `main` during the 2026-09-19 source review:

| Path | Git blob SHA |
|---|---|
| `src/dashboard/app_pro.py` | `5bbddc117748fd1341b8b6f5e0a026aa3316a6c2` |
| `src/dashboard/app_hardened.py` | `117aafcd421e9eb1b42cf360471aa65acaf5f98a` |
| `src/dashboard/safety.py` | `d9fb36c53ea639097d4e82ea623d3695a6197fc0` |
| `src/dashboard/derivatives.py` | `0da60f7f49c5a5e7888923c41bd0fe37d7deaa49` |
| `src/dashboard/conversion.py` | `b4a87a6812a5e3359eb66d786fe2494c2fdfd54e` |
| `src/dashboard/basket.py` | `07bf7df4a379b739d9cea49b7dd565975cb10d7f` |
| `src/dashboard/position_reconciliation.py` | `9a4fb1efa248b6dc230ea27cf75a1deeb4fccaaa` |
| `src/dashboard/silver_automation.py` | `8f129e333fa218d47b1ac4efc0970c0121daf3da` |
| `src/dashboard/silver_automation_ui.py` | `cac399cf3d7ae313cb399cff1985815823438d06` |
| `src/dashboard/templates/index.html` | `2b265bdd13e520fed17b32064aed03dc4bd5402c` |
| `src/dashboard/templates/derivatives.html` | `7b122cd33860cf7eb1749a29df4bc9372e8160f2` |
| `scripts/start_dashboard.sh` | `e3d0e5a968bf0edc2ebda2b3a3d774acef962ebe` |
| `scripts/status_dashboard.sh` | `880140488d18ab269afa2fff0a70ebee793dafb2` |
| `scripts/stop_dashboard.sh` | `05655dce34d59c3b1b567b251b9087e89b03ce32` |
| `scripts/restart_dashboard.sh` | `ec9416397f82a92c65c73a4d88e74d41b3b793b1` |

These are authoritative GitHub source identities, not yet a claim that the current OCI working tree contains those exact bytes.

## Startup contract

`scripts/start_dashboard.sh`:

1. targets `/home/ubuntu/kotak`;
2. starts one Uvicorn worker with `src.dashboard.app_pro:app`;
3. binds only to `127.0.0.1:8080`;
4. stops legacy Cloudflare dashboard tunnel processes;
5. verifies Tailscale is `Running`;
6. verifies the exact local DNS name `etf-trader.tailabfd53.ts.net.`;
7. verifies Serve is tailnet-only;
8. verifies Serve proxies to `http://127.0.0.1:8080`;
9. fails closed if private ingress is unhealthy.

## Security boundary

The dashboard can contain live broker capabilities behind server-side feature gates. This must not be confused with the NIFTY options paper-research engine, which has no broker order authorization.

The network architecture is also different:

```text
D01/D02 paper dashboards:
public Cloudflare Quick Tunnel -> chartink-paper:8765

D03 live ETF/Kotak dashboard:
private Tailscale Serve -> etf-trader/lakshmidevi:8080
```

## Remaining runtime proof

Before marking D03 `PROVEN_BYTE_IDENTICAL`, verify directly on `141.148.219.153`:

- hostname and Tailscale node identity;
- current checkout remote/branch/HEAD;
- current remote `main` SHA via read-only `git ls-remote`;
- working-tree status;
- actual Uvicorn PID/cwd/cmdline;
- Tailscale Serve route;
- all tracked `src/dashboard/` files against deployed HEAD;
- critical source Git blobs against current GitHub;
- no public Cloudflare process;
- local `/api/health` response.

A read-only Antigravity audit was dispatched as:

```text
DASHBOARD-ETF-RUNTIME-VERIFY-20260919-002
```

Until it returns, the correct confidence is:

```text
SOURCE + NETWORK ARCHITECTURE PROVEN
CURRENT DEPLOYED BYTE EQUIVALENCE PENDING
```

## Data and recovery boundary

Core live ETF strategy state is cloud/external-authoritative:

```text
AWS DynamoDB -> strategy quantities/BaseValue/campaign state + P&L lots
OCI Vault    -> secrets
GitHub       -> code/bootstrap/schedules
Kotak        -> actual broker positions/orders
Tailscale    -> network identity/grants
```

The host also has local operational state/config:

```text
/home/ubuntu/kotak/data/session_store.json
/home/ubuntu/kotak/data/dashboard_runtime.env
/home/ubuntu/kotak/data/dashboard_config.json
/home/ubuntu/kotak/data/logs/
/home/ubuntu/kotak/data/research/
```

A reboot normally preserves the boot disk. Total VM loss does not preserve all
of those local files unless they are recreated or separately backed up.

The older "zero state on disk" disaster-recovery statement in
`etf-invest-engine` is now being corrected to mean **core live ETF strategy
state is not disk-dependent**, not that the entire server is stateless.

See [MASTER_SYSTEM_MAP.md](MASTER_SYSTEM_MAP.md).
