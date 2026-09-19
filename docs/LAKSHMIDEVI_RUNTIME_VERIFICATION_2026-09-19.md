# lakshmidevi Runtime Verification — 2026-09-19

**Server:** `lakshmidevi`  
**Public OCI IP:** `141.148.219.153`  
**Tailscale node:** `etf-trader`  
**Observed Tailscale IPv4:** `100.120.194.42`  
**Audit source:** direct read-only shell output supplied from the running server on 2026-09-19.

No trade, service restart, configuration change, Git fetch/pull/reset, Tailscale change, Cloudflare change, deletion, move, or cleanup was performed.

## 1. Network and ingress

Observed listeners:

```text
0.0.0.0:22              sshd
127.0.0.1:8080          uvicorn
100.120.194.42:443      tailscaled / Tailscale Serve
```

Observed Tailscale Serve mapping:

```text
https://etf-trader.tailabfd53.ts.net (tailnet only)
|-- / proxy http://127.0.0.1:8080
```

No `cloudflared` process or service was observed on lakshmidevi.

Host firewall evidence showed SSH explicitly allowed, loopback/Tailscale traffic
allowed, and other new inbound traffic rejected after the Tailscale chains.
OCI Security List/NSG state was not independently proven by this host audit.

## 2. D03 — root ETF/Kotak operator dashboard

Observed process:

```text
/home/ubuntu/.venvs/dashboard/bin/uvicorn
src.dashboard.app_pro:app
--host 127.0.0.1
--port 8080
--workers 1
```

The root page identified itself as:

```text
Kotak Neo | Live Trading & Portfolio Dashboard
```

The process was running from:

```text
cwd: /home/ubuntu/kotak
```

### Startup ownership

At verification time the dashboard PID was in:

```text
/user.slice/user-1001.slice/session-767.scope
```

The journal showed an SSH login created that session immediately before the
dashboard process appeared. No matching dashboard system-level systemd service,
user-level systemd service, `@reboot` cron entry, or shell startup hook was
found.

Conclusion:

```text
D03 network ingress: private / proven
D03 current process: live / proven
D03 reboot autostart: NOT ESTABLISHED
```

Tailscale Serve can survive a reboot while the Uvicorn origin is absent.

## 3. D04 — `/v2/`

The route is present and returns a different frontend from `/`.

Observed response:

```html
<title>Trading Platform</title>
<script type="module" crossorigin src="/v2/assets/index-CDny3836.js"></script>
<link rel="stylesheet" crossorigin href="/v2/assets/index-BquxAb8W.css">
```

Observed source ownership:

```text
/home/ubuntu/kotak/src/dashboard/app_pro.py
  -> app.mount("/v2", StaticFiles(directory=str(WEB_DIST), html=True), ...)

/home/ubuntu/kotak/web/dist/index.html
/home/ubuntu/kotak/web/dist/assets/index-CDny3836.js
/home/ubuntu/kotak/web/dist/assets/index-BquxAb8W.css
```

The same FastAPI application also includes Platform-v2 API routers.

Conclusion:

```text
D04 is a distinct compiled frontend mounted in the same D03 FastAPI/Uvicorn process.
D04 is not a second strategy engine.
D04 is not merely a second Tailscale proxy mapping.
```

Do not remove it as a presumed duplicate until D03/D04 feature coverage and
operator usage are compared.

## 4. ETF checkout actually deployed

Observed:

```text
path: /home/ubuntu/kotak
branch: platform-v2-v03-preimplementation-20260916
HEAD: 306bd5c20dc48c682dab8c85ce1b2c677f97f66a
working tree: clean
server-local origin/main: b1485952fd9dc5846629c59134bfda9bb9765c21
HEAD...origin/main: 81 ahead, 0 behind
```

Current GitHub `main` was independently verified on 2026-09-19 at:

```text
56a5a1bb666ed76bd1a0ab253c897eda324547c3
```

The server audit deliberately did not fetch.

## 5. Weekday branch-reset risk

The `ubuntu` crontab contains a weekday 08:57 IST deployment step that:

```text
git fetch origin
git reset --hard origin/main
scripts/install_kotak_market_data_services.sh
```

Therefore the current feature/preimplementation checkout is not guaranteed to
remain on disk after the next weekday deployment job.

This is not a cleanup recommendation. It is a deployment-state discrepancy that
must be intentionally reconciled before branch normalization or other cleanup.

## 6. Hedge shadow control plane on lakshmidevi

A second Git checkout exists:

```text
/opt/hedge-engine
branch: silver-shadow-control-plane-20260915
HEAD: 70e82f25af6753f1b8b94fb776e16abe887b1053
working tree: clean
```

Observed service:

```text
hedge-engine-shadow.service
enabled
active
Restart=always
WorkingDirectory=/opt/hedge-engine
ExecStart -> python -m hedge_engine.runtime.shadow_loop
```

The service started automatically after reboot and writes mutable state under
`/var/lib/hedge-engine`.

At the audit timestamp it repeatedly reported:

```text
mode=SHADOW
broker=AUTH_FAILED
quote=UNKNOWN
recon=BROKER_UNAVAILABLE
phase=BROKER_UNAVAILABLE
```

That is operational-health evidence, not strategy authorization or a reason to
change hedge logic during this audit.

## 7. Local mutable data

Significant lakshmidevi-local files observed:

```text
/home/ubuntu/kotak/data/research/research-2026-07-*.sqlite3
/home/ubuntu/kotak/data/scrip_master.db
/home/ubuntu/kotak/data/platform_v2_operational.db
/home/ubuntu/kotak/data/session_store.json
/home/ubuntu/kotak/data/dashboard_config.json
/home/ubuntu/kotak/data/logs/dashboard_app.log
/home/ubuntu/kotak/data/*probe*.json

/var/lib/hedge-engine/state/live_strategy_state.json
/var/lib/hedge-engine/state/operator_control.json
/var/lib/hedge-engine/state/shadow_status.json
/var/lib/hedge-engine/data/dashboard_price_history.jsonl
/var/lib/hedge-engine/data/mcx_scrip_master.csv
/var/lib/hedge-engine/control/results/*.json
```

The ETF research directory contains multiple daily SQLite files in the
~140–236 MB range.

## 8. External durable authorities

Code/runtime evidence continues to support:

| State | Authority |
|---|---|
| ETF BaseValue / DefaultQuantity / AdditionalQuantity / campaign state | AWS DynamoDB |
| Campaign lot/P&L state | AWS DynamoDB |
| Actual positions/orders/holdings | Kotak broker |
| Secrets | OCI Vault |
| Source code | GitHub |
| Tailscale grants/identity | Tailscale control plane |
| Platform-v2 operational SQLite | lakshmidevi boot disk unless separately backed up |
| ETF research SQLite | lakshmidevi boot disk unless separately backed up |
| Hedge shadow state/control data | lakshmidevi boot disk unless separately backed up |

## 9. Backup evidence

The audit found one obvious historical local archive:

```text
/home/ubuntu/kotak-backup-20260601-061813.tar.gz
```

No current scheduled off-VM application backup/export was found in the user's
crontab or custom systemd units.

Repository backup code/runbooks are capability, not proof of an active off-VM
backup.

## 10. Durability summary

| System/data | Reboot-safe | VM-loss-safe | Current evidence |
|---|---:|---:|---|
| ETF DynamoDB strategy state | Yes | Yes | external AWS authority |
| Broker positions/orders | Yes | Yes | Kotak authority; reconcile |
| OCI secrets | Yes | Yes | external OCI authority |
| Git source | Yes | Yes | GitHub |
| Dashboard process | No | No | manual SSH-session launch; no autostart found |
| Dashboard local config/logs | Yes | No evidence | boot disk |
| Platform-v2 operational DB | Yes | No evidence | boot disk |
| ETF research SQLite | Yes | No evidence | boot disk |
| Hedge shadow local state | Yes | No evidence | boot disk |
| Tailscale Serve mapping | normal reboot expected if host state survives | replacement requires controlled reenrollment/reconfiguration | external + host state |

## 11. Cleanup status

Do not delete, rename, move, reset branches, change cron, change Tailscale,
restart services, or retire `/v2/` based only on this audit.

Before cleanup:

1. reconcile intended production branch versus observed deployed branch;
2. establish deliberate dashboard reboot persistence;
3. define off-VM backup/restore policy for local operational/evidence data;
4. compare D03 and D04 feature coverage;
5. complete the remaining chartink-paper security/durability follow-up.
