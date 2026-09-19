# Dashboard Server Topology

This document records the physical/logical server placement of the known dashboard surfaces. It separates **server identity**, **ingress**, **serving process**, and **generator ownership**.

## Two distinct OCI servers

The current documentation identifies two different OCI VMs:

| Server role | Host identity | Public IP | Other known IP | Primary dashboard responsibility |
|---|---|---:|---|---|
| Paper/research Server 2 | `chartink-paper` | `80.225.234.65` | OCI private `10.0.0.71` | NIFTY paper research + MCX/Silver paper research serving/publishing |
| Live ETF/Kotak server | `lakshmidevi` / Tailscale node `etf-trader` | `141.148.219.153` | Tailscale `100.120.194.42` at 2026-09-15 verification | Private ETF/Kotak live operator dashboard |

These are **not the same VM**.

The NIFTY Server-2 inventory explicitly states that `141.148.219.153` is outside the NIFTY paper-research project. The ETF platform documentation independently identifies `141.148.219.153 / lakshmidevi / etf-trader` as the live dashboard host.

## Server 2 — chartink-paper

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

- `docs/reports/SECOND_SERVER_BUILD_REPORT.md`
- `nifty-options-paper-research/docs/SERVER2_INVENTORY.md`

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
Tailscale IPv4:     100.120.194.42   (at 2026-09-15 verification)
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
runtime checkout documented as:
/home/ubuntu/kotak
     |
GitHub:
kriskingg/etf-invest-engine
main
```

The startup script explicitly validates the exact Tailscale node name and refuses to recreate a public Cloudflare fallback.

Source ownership and network architecture are strongly documented. The
`etf-invest-engine/main` branch is also the live ETF/MTF production strategy
branch, including Pair 1/2/3, KnowYourPNL, DynamoDB campaign integration and
cron. A fresh byte-for-byte server verification is required before this registry
claims the current deployed working tree exactly matches the current GitHub
`main` HEAD.

For ports, persistence, restart behavior, Internet exposure, backup gaps and
cleanup candidates, see [MASTER_SYSTEM_MAP.md](MASTER_SYSTEM_MAP.md).

### D04 — `/v2/`

Known URL:

```text
https://etf-trader.tailabfd53.ts.net/v2/
```

Its exact current route ownership must be established from the running `etf-trader` server rather than assumed from the common hostname. The runtime verification job must determine whether `/v2/` is:

- a FastAPI route in the same application;
- a Tailscale Serve path-specific proxy;
- a static/legacy route;
- a redirect/alias;
- or no longer present.

Until that runtime route trace is complete, D04 remains independently marked **PARTIAL/UNMAPPED**.

## Cross-server relationship

```text
SERVER 2 / PAPER RESEARCH
chartink-paper
80.225.234.65
  |
  +-- D01 NIFTY /live_latest.html
  +-- D02 MCX   /mcx_latest.html

LIVE ETF/KOTAK
lakshmidevi / etf-trader
141.148.219.153
Tailscale 100.120.194.42 (verified 2026-09-15)
  |
  +-- D03 /
  +-- D04 /v2/  [exact route owner pending runtime proof]
```

The two groups must not be treated as one server merely because source code in `kriskingg/etf-invest-engine` references both systems.

## Runtime verification jobs

Two read-only Antigravity audits were dispatched on 2026-09-19:

```text
DASHBOARD-RUNTIME-VERIFY-20260919-001
Target: 80.225.234.65 / chartink-paper
Scope: D01 + D02, full request path and byte-equivalence audit

DASHBOARD-ETF-RUNTIME-VERIFY-20260919-002
Target: 141.148.219.153 / lakshmidevi / etf-trader
Scope: D03 + D04, Tailscale route mapping and byte-equivalence audit
```

The audits are read-only. They are specifically forbidden from pulling code, restarting services, changing Tailscale/Cloudflare state, or calling broker order endpoints.

Do not promote a surface to `PROVEN_BYTE_IDENTICAL` until its corresponding runtime audit has returned actual deployed-byte evidence.