# Master Dashboard, Server, Data and Recovery Map

**Last consolidated:** 2026-09-19

This is the single operational map for the four dashboard URLs currently in use or under investigation. It separates **URL**, **network exposure**, **server**, **port**, **serving process**, **generator/source code**, **persistent data**, **reboot behavior**, **server-loss behavior**, and **cleanup status**.

## Four URLs, two OCI hosts, three known application owners

| ID | URL | Purpose | Server | External access | Local origin | Generator/source owner | Current confidence |
|---|---|---|---|---|---|---|---|
| D01 | `https://triumph-events-chair-problems.trycloudflare.com/live_latest.html` | NIFTY options paper research | `chartink-paper` / `80.225.234.65` | Cloudflare Quick Tunnel, Internet-facing URL | `127.0.0.1:8765` | `kriskingg/etf-invest-engine`, branch `chatgpt/statistical-options-buying-basket-v1` | Runtime source proven current; checkout metadata hybrid/stale |
| D02 | `https://triumph-events-chair-problems.trycloudflare.com/mcx_latest.html` | MCX / Silver hedge paper research | `chartink-paper` / `80.225.234.65` | same Cloudflare Quick Tunnel | `127.0.0.1:8765` | `kriskingg/hedge-engine`, `main` | Proven current runtime |
| D03 | `https://etf-trader.tailabfd53.ts.net/` | Live ETF/Kotak operator dashboard | `lakshmidevi / etf-trader` / `141.148.219.153` | Tailscale Serve, tailnet-only | `127.0.0.1:8080` | `kriskingg/etf-invest-engine`; observed deployed branch `platform-v2-v03-preimplementation-20260916` | Live runtime/ingress proven; deployed checkout differs from intended `main` authority |
| D04 | `https://etf-trader.tailabfd53.ts.net/v2/` | Platform-v2 compiled ETF/Kotak frontend | same `lakshmidevi` server | same private Tailscale hostname | same `127.0.0.1:8080` Uvicorn process | same FastAPI app; static frontend from `/home/ubuntu/kotak/web/dist` | **ROUTE OWNERSHIP PROVEN** |

The four URLs do **not** represent four independent strategy engines.

## chartink-paper — research host

```text
Hostname:        chartink-paper
Public IP:       80.225.234.65
Private IP:      10.0.0.71
OS observed:     Ubuntu 24.04.4 LTS
```

### Ingress and serving

Observed 2026-09-19:

```text
cloudflared
  -> http://127.0.0.1:8765
  -> nifty-multi-shadow-dashboard.service
  -> mcx_paper.control_server
  -> /var/lib/chartink-paper/nifty-multi-shadow/reports/
       |- live_latest.html
       |- mcx_latest.html
```

Port ownership:

| Port | Bind | Purpose |
|---|---|---|
| 8765 | `127.0.0.1` only | shared local NIFTY/MCX HTTP/control server |
| public HTTPS | Cloudflare-managed | Quick Tunnel endpoint forwarding to 8765 |

The common HTTP/control server is **not** proof of common generator ownership.

### D01 NIFTY generator

```text
nifty-multi-shadow-live.service / timer
  -> /opt/chartink-paper/nifty-options-v2
  -> analysis/nifty_multi_shadow/live.py
  -> analysis/nifty_multi_shadow/dashboard.py
  -> /var/lib/chartink-paper/nifty-multi-shadow/reports/live_latest.html
```

Source authority:

```text
kriskingg/etf-invest-engine
branch: chatgpt/statistical-options-buying-basket-v1
current GitHub HEAD verified: f9a8fe8a22173fcda902bcbc2f8fd3a4da4defa5
```

Deployment-layout caveat:

```text
server checkout HEAD: 97f51a865a71bd44ddadd2007f609b4c083556a0
runtime dashboard.py: matches current GitHub blob a6757e...
```

The executable runtime source is current for the changed application file, but the checkout metadata does not cleanly describe the runtime tree. Treat this as a hybrid deployment. Do not use destructive Git cleanup.

### D02 MCX generator

```text
hedge-engine-paper.service
  -> /opt/hedge-engine
  -> python -m hedge_engine.runtime.paper_loop
  -> /var/lib/hedge-engine/reports/mcx_latest.html
  -> publishes identical copy to:
     /var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
```

Source authority:

```text
kriskingg/hedge-engine
branch: main
deployed/GitHub HEAD:
e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071
```

Every tracked runtime file matched the deployed Git HEAD during the 2026-09-19 audit. The installed systemd unit differs from the repository copy only by CRLF versus LF line endings.

The historical `mcx-paper-live.service` exists but is disabled/inactive.

## lakshmidevi — live ETF/Kotak host

Freshly verified identity/runtime on 2026-09-19:

```text
Host:            lakshmidevi
Tailscale node:  etf-trader
Public OCI IP:   141.148.219.153
MagicDNS:        etf-trader.tailabfd53.ts.net
Tailscale IPv4:  100.120.194.42
Checkout:        /home/ubuntu/kotak
Observed branch: platform-v2-v03-preimplementation-20260916
Observed HEAD:   306bd5c20dc48c682dab8c85ce1b2c677f97f66a
```

This is the live ETF/MTF production host and is a different OCI VM from `chartink-paper`.

### D03 ETF/Kotak dashboard

Documented production architecture:

```text
approved Tailscale device
  -> HTTPS TCP 443
  -> Tailscale Serve (tailnet only)
  -> http://127.0.0.1:8080
  -> Uvicorn / FastAPI
  -> src.dashboard.app_pro:app
  -> /home/ubuntu/kotak
  -> kriskingg/etf-invest-engine
  -> main
```

The `main` branch is not merely a dashboard branch. It is the repository's **live ETF/MTF production branch**, including the scheduled ETF strategy.

However, the fresh host audit found the actual checkout on
`platform-v2-v03-preimplementation-20260916` at `306bd5c...`, with a stale
server-local `origin/main` ref. The weekday 08:57 IST deployment cron fetches
and hard-resets the checkout to `origin/main`. Treat this as a deployment-state
mismatch requiring intentional reconciliation; do not assume the next reset is a
no-op.

Known live components on `main`:

```text
pair_1/                  login + holdings synchronization
pair_2/main_kotak.py     scheduled default MTF buy
pair_3/kotak_price_drop.py
                         qualifying price-drop MTF buys
KnowYourPNL/             campaign lot/cost/P&L accounting
AWS DynamoDB             campaign strategy + lot state
cron                     scheduled live workflow
src/dashboard/           private live operator dashboard
```

D03 must therefore be treated as a production operator surface, not as a research dashboard.

### D04 /v2/

Fresh 2026-09-19 runtime evidence proved D04 is a distinct compiled frontend
mounted inside the same FastAPI/Uvicorn application as D03:

```text
src/dashboard/app_pro.py
  -> app.mount("/v2", StaticFiles(directory=str(WEB_DIST), html=True))

WEB_DIST:
  /home/ubuntu/kotak/web/dist
```

Observed assets:

```text
web/dist/index.html
web/dist/assets/index-CDny3836.js
web/dist/assets/index-BquxAb8W.css
```

D04 is therefore not a second strategy engine and not a separate Tailscale path
proxy. It may still overlap D03 functionally, but feature coverage and operator
usage must be compared before any retirement decision.

See [D04-etf-dashboard-v2.md](D04-etf-dashboard-v2.md).

## Internet visibility and access

### D01/D02 research URLs

The Cloudflare Quick Tunnel hostname is an Internet-facing endpoint. The local service is loopback-only, but Cloudflare intentionally bridges the public URL to that loopback port.

Current conclusion:

```text
D01/D02: treat as publicly reachable by URL unless authentication/access
controls are separately proven.
```

The serving process also has a paper-control argument:

```text
--control /var/lib/chartink-paper/mcx-paper/control.json
```

Fresh source audit proved GET and POST on `/api/mcx-paper-control`. POST mutates the control file and is protected only by a fixed `X-Paper-Control: local-dashboard` header plus an Origin check; no secret/token authentication was detected. Because the same Quick Tunnel exposes this HTTP origin publicly, treat the control surface as a **security remediation blocker**. No mutating request was executed during the audit.

### D03/D04 ETF URLs

Repository design and 2026-09-15 verification state:

```text
Tailscale Serve: tailnet only
Uvicorn: 127.0.0.1:8080 only
Tailscale Funnel: not used
Cloudflare dashboard tunnel: disabled
Device Approval: enabled
Application authentication: required
```

Therefore D03 is designed to be private, not visible to the general Internet.

D04 is proven to inherit the same tailnet-only Tailscale/Uvicorn boundary because it is mounted inside the same application process.

## Data ownership and durability

### Live ETF/MTF production state

Authoritative durable strategy/accounting state documented in `etf-invest-engine/main`:

| Data | Authority | Survives VM loss? |
|---|---|---|
| ETF strategy configuration, quantities, BaseValue | AWS DynamoDB active campaign Alpha table | Yes, cloud state |
| Campaign P&L/lots | AWS DynamoDB KnowYourPNL table | Yes, cloud state |
| Secrets | OCI Vault | Yes, cloud state |
| Source code, schedules, bootstrap | GitHub | Yes |
| Current broker positions/orders | Kotak Neo broker | Yes; must be re-read/reconciled |
| Tailscale grants/device state | Tailscale control plane | external state |

Host-local production/runtime files also exist and must not be described as "zero state":

```text
/home/ubuntu/kotak/data/session_store.json
/home/ubuntu/kotak/data/dashboard_runtime.env
/home/ubuntu/kotak/data/dashboard_config.json
/home/ubuntu/kotak/data/platform_v2_operational.db
/home/ubuntu/kotak/data/logs/
/home/ubuntu/kotak/data/research/
/var/lib/hedge-engine/        shadow state/control/evidence on lakshmidevi
pair_1/load_secrets.sh        generated runtime file
```

Some are reproducible; some contain local operational state/config and require deliberate recovery handling.

### ETF-server research data

Main-branch market-data research writes daily SQLite files:

```text
/home/ubuntu/kotak/data/research/research-YYYY-MM-DD.sqlite3
```

Repository operations documentation currently describes bounded retention. These files are local-disk evidence and are **not automatically protected merely because the live ETF strategy state is in DynamoDB**.

### NIFTY paper-research data

Logical runtime roots include:

```text
/var/lib/chartink-paper/nifty-options-v2/
/var/lib/chartink-paper/nifty-multi-shadow/
/var/lib/chartink-paper/evidence-repo/
/var/lib/chartink-paper/mcx-paper/
```

A final-gap audit run as `ubuntu` showed `/var/lib/chartink-paper` is mode
`0700`, owned by `chartink-paper`. The `ubuntu` account therefore could
not traverse the NIFTY runtime tree; the earlier 4 KiB result was an
**audit-permission artifact**, not evidence of symlinks or missing data. A
complete NIFTY mutable-data inventory must be collected as root or the
`chartink-paper` service user if/when durability closure is required.

Git protects source and the evidence exporter protects selected COMPLETE packets.
The exporter is currently unhealthy for several recent sessions, so recent
off-VM evidence coverage must not be assumed.

### MCX hedge-engine data

Known local roots:

```text
/var/lib/hedge-engine/data/
/var/lib/hedge-engine/state/
/var/lib/hedge-engine/reports/
/var/lib/hedge-engine/executions/
```

Git protects source code, not these mutable runtime directories. A separate off-server backup/restore contract is not yet proven in this registry.

## Restart versus disaster recovery

A normal reboot and total VM loss are different events.

### chartink-paper reboot

Known:
- `nifty-multi-shadow-dashboard.service` is enabled;
- `hedge-engine-paper.service` is enabled and configured `Restart=always`;
- NIFTY live engine is timer-triggered;
- old `mcx-paper-live.service` is disabled.

Freshly proven:
- `cloudflared-tunnel.service` is enabled, active and `Restart=always`;
- it launches `cloudflared tunnel --url http://127.0.0.1:8765`;
- this is a Cloudflare Quick Tunnel, which Cloudflare documents as generating a
  random `trycloudflare.com` subdomain and being intended for
  testing/development.

Therefore cloudflared itself is configured to return after reboot, but the
current Quick Tunnel hostname is **not a durable address** and should be expected
to change when a fresh cloudflared process creates a new Quick Tunnel.

### lakshmidevi reboot

Fresh 2026-09-19 evidence:

- Tailscale Serve is active and tailnet-only, proxying to `127.0.0.1:8080`;
- the current Uvicorn dashboard process was launched from an SSH session and is
  not owned by an observed dashboard systemd service, user service,
  `@reboot` cron entry, or shell startup hook;
- D04 is mounted inside that same Uvicorn process;
- `hedge-engine-shadow.service` is enabled at boot and uses
  `Restart=always`.

Therefore a normal reboot can restore Tailscale while leaving D03/D04
unavailable until Uvicorn is deliberately started. Dashboard reboot persistence
is a confirmed gap, not a pending question.

### Total server loss

GitHub can restore code. It cannot restore every mutable local file.

Cloud-authoritative ETF state has much stronger disaster recovery because strategy state/P&L is in DynamoDB and secrets are in OCI Vault.

Local research SQLite, local dashboard configuration, NIFTY/MCX raw/runtime files and generated artifacts require separate treatment. The existing `etf-invest-engine/docs/DISASTER_RECOVERY.md` statement that the server has "zero state on disk" is too broad and must be read as referring to **core live ETF strategy state**, not all operational/research state.

## Are we implementing the same thing multiple times?

Some duplication is intentional; some is historical.

### Intentional separation

- NIFTY research and MCX hedge research are independent paper-research engines.
- The ETF dashboard is a live operator surface and must not be merged blindly with research execution logic.
- `hedge-engine` is the authoritative MCX strategy implementation; the ETF dashboard may display/reference its state but should not become a second independent MCX strategy engine.

### Confirmed/higher-risk duplication or residue

1. `mcx_paper.control_server` / `/opt/chartink-paper/mcx-paper-research` still serves the shared research pages even after MCX generation moved to `hedge-engine`.
2. `nifty-multi-shadow-dashboard.service` name is misleading because it serves both NIFTY and MCX.
3. historical `mcx-paper-live.service` remains installed but disabled.
4. NIFTY uses a hybrid untracked runtime tree while its checkout HEAD is old.
5. `/v2/` is proven to be a distinct frontend in the same application; functional overlap with D03 remains to be assessed.
6. `etf-invest-engine` contains both the live ETF production system on `main` and separate NIFTY research work on other branches, making repo-name-only ownership ambiguous.
7. two untracked hedge-engine deployment tarballs were observed on `chartink-paper`.

## Cleanup policy

Do **not** delete, rename, consolidate or move any component yet.

Cleanup is allowed only after dependency proof.

### Phase 1 — complete mapping

Pending:
- reconcile intended live ETF production branch with the observed
  `platform-v2-v03-preimplementation-20260916` checkout and weekday hard reset
  to `origin/main`;
- design/test deliberate D03/D04 reboot autostart;
- compare D03 and D04 feature coverage before any route retirement;
- remediate/separate the Internet-facing chartink-paper control endpoint;
- complete the NIFTY mutable-data inventory using root/`chartink-paper` read access (the prior `ubuntu` audit could not traverse the 0700 parent);
- classify/fix or formally retire the two broken scheduled NIFTY replay paths;
- repair recent forward-evidence delivery;
- verify OCI boot-volume backup policy;
- complete off-server backup/restore matrix and restore testing for all local mutable data.

### Phase 2 — dependency graph

For every suspected legacy component, identify:
- current process/service/timer/cron references;
- file writers and readers;
- dashboard links/API consumers;
- rollback dependency;
- last-use evidence.

## Dashboard code/repository decision control

The canonical implementation map is
[DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md](DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md).
The repository-boundary register is
[DASHBOARD_REPOSITORY_BOUNDARY_DECISIONS.md](DASHBOARD_REPOSITORY_BOUNDARY_DECISIONS.md).

Current architecture direction:

```text
Dashboards repo              -> documentation / ownership / decisions only
NIFTY paper research         -> candidate dedicated repo
MCX/Silver hedge + D02       -> keep hedge-engine
Live ETF/Kotak + D03 backend -> keep etf-invest-engine
Platform-v2 backend          -> keep ETF domain for now
Platform-v2 React frontend   -> optional later split after stable API
D01/D02 shared HTTP serving  -> candidate small read-only gateway
MCX mutating control         -> remove from public shared origin / keep domain-private
```

The objective is not one repo per URL. It is one unambiguous domain authority
with no duplicated safety/business logic.

### Phase 3 — proposed target architecture

Desired end state:

```text
Research server
  NIFTY -> one generator, one code authority, one data/recovery contract
  MCX   -> one generator, one code authority, one data/recovery contract
  shared serving layer only if intentionally retained

Live ETF server
  ETF strategy -> one production branch and one state/recovery contract
  operator dashboard -> one private ingress and one application
  /v2/ -> either documented role or retired after proof
```

No runtime cleanup should occur during market hours or without explicit approval.