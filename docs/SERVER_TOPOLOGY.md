# Dashboard Server Topology

This document records the physical/logical server placement of the known dashboard surfaces. It separates **server identity**, **ingress**, **serving process**, and **generator ownership**.

## Two distinct OCI hosts

The current documentation identifies two different OCI VMs. Current terminology uses hostnames, not ordinal labels such as "server 1" or "server 2":

| Server role | Host identity | Public IP | Other known IP | Primary dashboard responsibility |
|---|---|---:|---|---|
| Research host | `chartink-paper` | `80.225.234.65` | OCI private `10.0.0.71` | NIFTY paper research + MCX/Silver paper research serving/publishing |
| Live ETF/Kotak server | `lakshmidevi` / Tailscale node `etf-trader` | `141.148.219.153` | Tailscale `100.120.194.42` at 2026-09-19 verification | Private ETF/Kotak live operator dashboard |

These are **not the same VM**.

The historical NIFTY inventory for `chartink-paper` explicitly states that `141.148.219.153` is outside the NIFTY paper-research project. The ETF platform documentation independently identifies `141.148.219.153 / lakshmidevi / etf-trader` as the live dashboard host.

## chartink-paper — research host

Verified build/inventory facts recorded in `kriskingg/etf-invest-engine`:

```text
Hostname:          chartink-paper
Public IPv4:       80.225.234.65
Private IPv4:      10.0.0.71
OCI region:        ap-mumbai-1
Availability AD:   WfQK:AP-MUMBAI-1-AD-1
Shape:             VM.Standard.E2.1.Micro
OS:                Ubuntu 24.04
```

Relevant source records:

- `docs/reports/SECOND_SERVER_BUILD_REPORT.md` *(historical filename; refers to `chartink-paper`)*
- `nifty-options-paper-research/docs/SERVER2_INVENTORY.md` *(historical filename; refers to `chartink-paper`)*

### D01 — NIFTY paper research

```text
https://triumph-events-chair-problems.trycloudflare.com/live_latest.html
     |
Cloudflare Quick Tunnel
     |
http://127.0.0.1:8765
     |
chartink-paper / 80.225.234.65
     |
shared local HTTP/control server
     |
/var/lib/chartink-paper/nifty-multi-shadow/reports/live_latest.html
     ^
     |
NIFTY generator:
  /opt/chartink-paper/nifty-options-v2
  analysis/nifty_multi_shadow/live.py
  analysis/nifty_multi_shadow/dashboard.py
     |
GitHub:
  kriskingg/etf-invest-engine
  branch chatgpt/statistical-options-buying-basket-v1
```

Fresh 2026-09-19 proof established that the executable runtime `dashboard.py` blob is exactly `a6757e3087e643b0ac75f41ce993e371b3aac35b`, matching the current GitHub branch head. The checkout HEAD itself remains older (`97f51a8...`), so the deployment is a hybrid layout: current executable runtime source under the untracked top-level `analysis/` tree, stale checkout metadata underneath.

### D02 — MCX / Silver hedge research

```text
https://triumph-events-chair-problems.trycloudflare.com/mcx_latest.html
     |
same Cloudflare Quick Tunnel
     |
http://127.0.0.1:8765
     |
chartink-paper / 80.225.234.65
     |
shared report directory
     |
/var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
     ^
     |
published by:
  /opt/hedge-engine
  python -m hedge_engine.runtime.paper_loop
     |
primary report:
  /var/lib/hedge-engine/reports/mcx_latest.html
     |
GitHub:
  kriskingg/hedge-engine
  branch main
```

The hedge-engine migration runbook explicitly says the migration procedure runs on OCI node `80.225.234.65`.

Fresh 2026-09-19 proof established that every Git-tracked file in `/opt/hedge-engine` matches deployed HEAD `e10eeeda...`, and GitHub `main` is at the same commit. Primary and published MCX HTML copies matched byte-for-byte. The installed systemd unit differs from the repository copy only by CRLF versus LF line endings.

## Live ETF/Kotak server — lakshmidevi / etf-trader

Current repository documentation identifies the private operator host as:

```text
Public OCI IP:      141.148.219.153
Host identity:      lakshmidevi
Tailscale hostname: etf-trader
MagicDNS:           etf-trader.tailabfd53.ts.net
Tailscale IPv4:     100.120.194.42   (verified 2026-09-19)
Origin:             http://127.0.0.1:8080
Ingress:            Tailscale Serve, tailnet-only HTTPS 443
Public Cloudflare:  disabled
```

Relevant source records:

- `docs/TAILSCALE_PRIVATE_DASHBOARD.md`
- `docs/STATUS.md`
- `docs/DASHBOARD_ARCHITECTURE.md`
- `docs/PLATFORM_VISION_AND_ROADMAP.md`
- `scripts/start_dashboard.sh`

### D03 — private LIVE ETF/Kotak dashboard root

Known source/runtime design:

```text
https://etf-trader.tailabfd53.ts.net/
     |
Tailscale Serve (tailnet only)
     |
http://127.0.0.1:8080
     |
FastAPI/Uvicorn
src.dashboard.app_pro:app
     |
runtime checkout observed as:
/home/ubuntu/kotak
branch platform-v2-v03-preimplementation-20260916
HEAD 306bd5c20dc48c682dab8c85ce1b2c677f97f66a
     |
GitHub repository:
kriskingg/etf-invest-engine
intended production branch: main
```

The startup script explicitly validates the exact Tailscale node name and refuses to recreate a public Cloudflare fallback.

Fresh 2026-09-19 runtime proof established the live Uvicorn process,
loopback binding, Tailscale Serve mapping, absence of cloudflared and exact
checkout branch/HEAD. The observed checkout does **not** match current GitHub
`main`: it is on `platform-v2-v03-preimplementation-20260916` at
`306bd5c...`. The weekday 08:57 IST deployment job fetches and hard-resets the
checkout to `origin/main`, so this branch discrepancy is an operational issue
to reconcile before cleanup.

For ports, persistence, restart behavior, Internet exposure, backup gaps and
cleanup candidates, see [MASTER_SYSTEM_MAP.md](MASTER_SYSTEM_MAP.md).

### D04 — `/v2/`

Fresh 2026-09-19 runtime proof established:

```text
https://etf-trader.tailabfd53.ts.net/v2/
     |
same Tailscale Serve
     |
same http://127.0.0.1:8080
     |
same src.dashboard.app_pro:app
     |
FastAPI StaticFiles mount
     |
/home/ubuntu/kotak/web/dist
```

D04 is a distinct compiled frontend inside the D03 application. It is not a
separate process, Tailscale route, or strategy engine.

Observed assets:

```text
web/dist/index.html
web/dist/assets/index-CDny3836.js
web/dist/assets/index-BquxAb8W.css
```

See [D04-etf-dashboard-v2.md](D04-etf-dashboard-v2.md).

## Cross-server relationship

```text
CHARTINK-PAPER / PAPER RESEARCH
chartink-paper
80.225.234.65
  |
  +-- D01 NIFTY /live_latest.html
  +-- D02 MCX   /mcx_latest.html

LIVE ETF/KOTAK
lakshmidevi / etf-trader
141.148.219.153
Tailscale 100.120.194.42 (verified 2026-09-19)
  |
  +-- D03 /      -> root live dashboard
  +-- D04 /v2/   -> distinct compiled frontend in same Uvicorn app
```

The two groups must not be treated as one server merely because source code in `kriskingg/etf-invest-engine` references both systems.

## Runtime verification status

Both hosts now have direct read-only runtime evidence recorded on 2026-09-19.

- `chartink-paper`: D01/D02 ownership and runtime provenance recorded in
  [CHARTINK_PAPER_RUNTIME_VERIFICATION_2026-09-19.md](CHARTINK_PAPER_RUNTIME_VERIFICATION_2026-09-19.md).
- `lakshmidevi`: D03/D04 route/process/checkout/durability evidence recorded in
  [LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md](LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md).

No audit authorized code pulls, service restarts, network changes, broker order
calls, cleanup or branch normalization.
