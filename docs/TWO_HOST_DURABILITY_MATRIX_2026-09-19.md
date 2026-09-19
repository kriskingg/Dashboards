# Two-Host Durability Matrix — 2026-09-19

**Scope:** `lakshmidevi` and `chartink-paper`  
**Status:** operationally useful, but not fully closed until the two remaining
chartink-paper durability inputs are resolved.

## Classification

- **Reboot-safe:** expected to survive a normal OS/VM reboot when the existing
  boot volume remains intact.
- **VM-loss-safe:** evidence exists that the data is authoritative or backed up
  outside the VM/boot volume.
- **Unknown:** the audit did not prove the condition.

## Lakshmidevi

| System/data | Writer/authority | Storage | Reboot-safe | VM-loss-safe | Backup/recovery evidence | Restore tested |
|---|---|---|---:|---:|---|---|
| ETF campaign BaseValue / default/additional quantity | ETF strategy | AWS DynamoDB | Yes | Yes | external AWS authority | application recovery flow exists; re-test pending |
| ETF campaign lot/P&L state | KnowYourPNL / ETF runtime | AWS DynamoDB | Yes | Yes | external AWS authority | re-test pending |
| Broker positions/orders/holdings | Kotak Neo | broker | Yes | Yes | broker is authority; must re-read/reconcile | operational reconciliation required |
| Secrets | OCI Vault | OCI | Yes | Yes | external secret authority | load/bootstrap flow exists |
| Source | GitHub | GitHub | Yes | Yes | external source authority | clone/bootstrap available |
| D03/D04 Uvicorn process | manual SSH-session launch observed | process only | **No proven autostart** | No | none | No |
| Tailscale Serve mapping | tailscaled + control plane | host + Tailscale | Yes when daemon/host state survive | replacement requires reenrollment/config | external control plane + host state | partial |
| Dashboard config/logs | dashboard | boot disk | Yes | No evidence | no scheduled off-VM backup found | No evidence |
| Platform-v2 operational SQLite | D04/runtime | boot disk | Yes | No evidence | local SQLite backup capability only | No evidence |
| ETF research SQLite | research recorder | boot disk | Yes | No evidence | no scheduled off-VM backup found | No evidence |
| Hedge shadow state/control/evidence | hedge shadow service | `/var/lib/hedge-engine` boot disk | Yes | No evidence | no scheduled off-VM backup found | No evidence |

## Chartink-paper — ingress / serving

| System/data | Writer/authority | Storage | Reboot-safe | VM-loss-safe | Backup/recovery evidence | Restore tested |
|---|---|---|---:|---:|---|---|
| cloudflared process | `cloudflared-tunnel.service` | systemd + process | Yes | source/unit recoverable, hostname not durable | unit enabled, `Restart=always` | reboot not deliberately tested in this audit |
| Quick Tunnel hostname | Cloudflare Quick Tunnel | Cloudflare temporary allocation | **No stable-hostname guarantee** | No | Cloudflare documents random temporary hostname | N/A |
| shared HTTP/control service | `nifty-multi-shadow-dashboard.service` / `mcx_paper.control_server` | boot-disk code + systemd | Yes when unit succeeds | code recoverable; runtime state separate | source/runtime mapping proven | rebuild not restore-tested |
| public NIFTY/MCX HTML pages | NIFTY/MCX generators | logical `/var/lib/chartink-paper/.../reports` | Yes if boot volume/runtime path survives | selected NIFTY evidence only; generated HTML reproducible | public HTTP 200 proven | N/A |

## Chartink-paper — NIFTY

| System/data | Writer/authority | Storage | Reboot-safe | VM-loss-safe | Backup/recovery evidence | Restore tested |
|---|---|---|---:|---:|---|---|
| NIFTY raw Kotak session SQLite | recorder | logical `/var/lib/chartink-paper/nifty-options-v2/data/research-YYYY-MM-DD.sqlite3` | likely yes on boot volume | **No active off-VM backup proven** | docs recommend Object Storage/laptop; no scheduled server job found | No evidence |
| NIFTY live paper state | live engine | logical `/var/lib/chartink-paper/nifty-multi-shadow/live/YYYY-MM-DD/state.json` | likely yes | partial: COMPLETE evidence packets published, raw state not fully protected | evidence branch for COMPLETE packets | No full replacement-VM drill proven |
| NIFTY reports | live/replay engines | logical `/var/lib/chartink-paper/nifty-multi-shadow/reports` | likely yes | generated artifacts partly reproducible | selected evidence in Git | No |
| forward evidence packets | evidence exporter | Git `paper-forward-evidence` | Yes | Yes for sessions proven COMPLETE + remotely published | remote publication verified for many older sessions | packet verification built in |
| Sep 8-18 recent evidence | evidence exporter | intended Git evidence branch | local/raw status only | **Not currently protected as COMPLETE evidence** | Sep 18 exporter logged 8 operational failures/no manifest | No |
| daily multi-shadow replay/report | `nifty-multi-shadow.service` | derived from raw SQLite | scheduled but **currently failing** | derived/reproducible if raw data survives | Sep 15-18 failed opening SQLite | No |
| older NIFTY V2 replay | `nifty-options-v2-replay.timer/service` | derived | **currently broken** | unknown/likely reproducible | Sep 15-18 `203/EXEC` | No |

**Inventory warning:** `/var/lib/chartink-paper` is mode `0700` and owned by
`chartink-paper`. The follow-up command was run as `ubuntu`, which could not
traverse the parent. Therefore the earlier 4 KiB result is not meaningful for
NIFTY data size and does not prove symlinked paths. A complete raw/state
inventory requires read-only root or `chartink-paper` execution.

## Chartink-paper — MCX / hedge-engine

| System/data | Writer/authority | Storage | Reboot-safe | VM-loss-safe | Backup/recovery evidence | Restore tested |
|---|---|---|---:|---:|---|---|
| MCX code | hedge-engine | GitHub | Yes | Yes | current tracked runtime proven | clone/build possible |
| runtime checkpoint | hedge-engine paper loop | `/var/lib/hedge-engine/state/runtime_checkpoint.json` | Yes | No off-VM evidence | same-disk snapshots exist | No |
| strategy state | hedge-engine paper loop | `/var/lib/hedge-engine/state/strategy_state.json` | Yes | No off-VM evidence | same-disk snapshots exist | No |
| ETF ledger | hedge-engine paper loop | `/var/lib/hedge-engine/state/etf_ledger.json` | Yes | No off-VM evidence | same-disk snapshots exist | No |
| runtime health | hedge-engine paper loop | `/var/lib/hedge-engine/state/runtime_health.json` | Yes | No off-VM evidence | generated | No |
| execution history | hedge-engine paper loop | `/var/lib/hedge-engine/executions/execution_history.jsonl` | Yes | No off-VM evidence | same-disk snapshots exist | No |
| price history | hedge-engine paper loop | `/var/lib/hedge-engine/data/dashboard_price_history.jsonl` | Yes | No off-VM evidence | same-disk snapshot exists | No |
| MCX scrip master | refresh/runtime | `/var/lib/hedge-engine/data/mcx_scrip_master.csv` | Yes | No, but refreshable | source can be reacquired | not material if refresh succeeds |
| MCX HTML report | paper loop | `/var/lib/hedge-engine/reports/mcx_latest.html` + published copy | Yes | generated/reproducible | primary/published bytes proven identical | N/A |
| local hedge backups | manual/predeploy tooling | `/var/lib/hedge-engine/backups` | Yes | **No** — same `/dev/sda1` failure domain | multiple snapshots observed | No full VM restore |

All audited chartink-paper application roots resolve to the VM root ext4
filesystem on `/dev/sda1`. Therefore local NIFTY/MCX mutable state should be
treated as boot-volume state unless external copies are specifically proven.

## Security/durability blockers before cleanup

1. The public Quick Tunnel shares an origin with a mutating
   `POST /api/mcx-paper-control` endpoint. Fixed header + Origin validation is
   not strong authentication.
2. Quick Tunnel hostname is temporary/random, not a durable DR endpoint.
3. Daily NIFTY multi-shadow replay currently fails to open its SQLite input.
4. NIFTY V2 replay timer triggers a service that exits `203/EXEC`.
5. Recent forward-evidence delivery is incomplete/failing.
6. Full NIFTY mutable-data inventory still needs privileged read-only traversal of the `0700` `/var/lib/chartink-paper` tree.
7. OCI boot-volume backup policy/existing backups are still unknown.
8. No tested replacement-VM restore was evidenced for raw NIFTY SQLite or MCX
   mutable runtime state.

No cleanup or runtime change is authorized by this matrix.
