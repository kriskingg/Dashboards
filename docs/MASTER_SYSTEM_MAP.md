# Master Dashboard, Server, Data and Recovery Map

**Last consolidated:** 2026-09-19

This is the single operational map for the four dashboard URLs currently in use or under investigation. It separates **URL**, **network exposure**, **server**, **port**, **serving process**, **generator/source code**, **persistent data**, **reboot behavior**, **server-loss behavior**, and **cleanup status**.

## Four URLs, two servers, three known application owners

| ID | URL | Purpose | Server | External access | Local origin | Generator/source owner | Current confidence |
|---|---|---|---|---|---|---|---|
| D01 | `https://triumph-events-chair-problems.trycloudflare.com/live_latest.html` | NIFTY options paper research | `chartink-paper` / `80.225.234.65` | Cloudflare Quick Tunnel, Internet-facing URL | `127.0.0.1:8765` | `kriskingg/etf-invest-engine`, branch `chatgpt/statistical-options-buying-basket-v1` | Runtime source proven current; checkout metadata hybrid/stale |
| D02 | `https://triumph-events-chair-problems.trycloudflare.com/mcx_latest.html` | MCX / Silver hedge paper research | `chartink-paper` / `80.225.234.65` | same Cloudflare Quick Tunnel | `127.0.0.1:8765` | `kriskingg/hedge-engine`, `main` | Proven current runtime |
| D03 | `https://etf-trader.tailabfd53.ts.net/` | Live ETF/Kotak operator dashboard | `lakshmidevi / etf-trader` / `141.148.219.153` | Tailscale Serve, tailnet-only | documented `127.0.0.1:8080` | `kriskingg/etf-invest-engine`, `main` | Source/network design documented; fresh runtime re-check pending |
| D04 | `https://etf-trader.tailabfd53.ts.net/v2/` | Unknown/legacy/alternate ETF dashboard route until proven | expected same ETF server | expected same private Tailscale hostname | not yet proven | not yet proven | **UNMAPPED** |

The four URLs do **not** represent four independent strategy engines.

## Server A — chartink-paper research server

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

## Server B — live ETF/Kotak server

Documented identity:

```text
Host:            lakshmidevi
Tailscale node:  etf-trader
Public OCI IP:   141.148.219.153
MagicDNS:        etf-trader.tailabfd53.ts.net
Tailscale IPv4:  100.120.194.42   (verified 2026-09-15)
Checkout:        /home/ubuntu/kotak
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

The hostname alone does not prove route ownership.

Still to determine from Server B:

```text
/v2/ is one of:
- same FastAPI application route;
- Tailscale Serve path mapping;
- redirect/alias;
- static/legacy page;
- separate process;
- obsolete/not present.
```

Do not create or preserve duplicate functionality merely because `/v2/` exists. First identify what it is.

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

The exact HTTP control endpoints and their authentication still need a targeted read-only security review. Do not assume "paper only" makes a publicly exposed control surface harmless.

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

D04 inherits only the hostname's private network boundary until its exact route/process is verified.

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
/home/ubuntu/kotak/data/logs/
/home/ubuntu/kotak/data/research/
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

Known local roots include:

```text
/var/lib/chartink-paper/nifty-options-v2/
/var/lib/chartink-paper/nifty-multi-shadow/
```

Git protects source; forward evidence publication protects selected completed evidence packets; raw recorder/runtime data on OCI still needs an explicit off-server recovery policy if it must survive total disk loss.

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

### Server A reboot

Known:
- `nifty-multi-shadow-dashboard.service` is enabled;
- `hedge-engine-paper.service` is enabled and configured `Restart=always`;
- NIFTY live engine is timer-triggered;
- old `mcx-paper-live.service` is disabled.

Still to prove:
- exactly how `cloudflared` is recreated after a full reboot;
- whether the Quick Tunnel retains or changes its hostname.

Do not treat the current Quick Tunnel URL as a durable disaster-recovery address.

### Server B reboot

Repository design says:
- live ETF strategy scheduling is cron/systemd based;
- Tailscale is the production private ingress;
- dashboard launcher is `scripts/start_dashboard.sh`;
- startup must fail closed rather than create a public fallback.

Fresh Server-B proof is still required for:
- exact reboot/autostart mechanism of the Uvicorn dashboard;
- current Tailscale Serve state after reboot;
- D04 route behavior.

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
5. `/v2/` may duplicate or obsolete part of D03; not yet proven.
6. `etf-invest-engine` contains both the live ETF production system on `main` and separate NIFTY research work on other branches, making repo-name-only ownership ambiguous.
7. two untracked hedge-engine deployment tarballs were observed on Server A.

## Cleanup policy

Do **not** delete, rename, consolidate or move any component yet.

Cleanup is allowed only after dependency proof.

### Phase 1 — complete mapping

Pending:
- fresh Server-B runtime/route audit;
- D04 ownership;
- Server-B restart/autostart proof;
- Cloudflare control-endpoint authentication review;
- off-server backup/restore matrix for all local mutable data.

### Phase 2 — dependency graph

For every suspected legacy component, identify:
- current process/service/timer/cron references;
- file writers and readers;
- dashboard links/API consumers;
- rollback dependency;
- last-use evidence.

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
