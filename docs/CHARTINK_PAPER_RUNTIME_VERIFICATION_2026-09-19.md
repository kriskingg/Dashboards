# chartink-paper Runtime Verification — 2026-09-19

**Server:** `chartink-paper`  
**Documented public IP:** `80.225.234.65`  
**Observed private IP:** `10.0.0.71`  
**OS:** Ubuntu 24.04.4 LTS  
**Kernel:** Linux 6.17.0-1019-oracle  
**Audit source:** direct read-only shell output supplied from the running server on 2026-09-19.

This document records the fresh runtime evidence for D01 NIFTY paper research and D02 MCX / Silver hedge research.

## 1. Server identity

Observed directly:

```text
hostname: chartink-paper
private IPv4: 10.0.0.71
OS: Ubuntu 24.04.4 LTS
kernel: 6.17.0-1019-oracle
architecture: x86-64
virtualization: KVM/QEMU
```

The public IP `80.225.234.65` remains the documented `chartink-paper` address from the historical build/inventory records. The command set used in this audit did not independently print the public IPv4.

## 2. Shared serving and Cloudflare ingress

The serving unit is active:

```text
nifty-multi-shadow-dashboard.service
status: active (running)
MainPID: 812870
```

Exact process:

```text
/opt/chartink-paper/mcx-paper-research/.venv/bin/python
-m mcx_paper.control_server
--bind 127.0.0.1
--port 8765
--directory /var/lib/chartink-paper/nifty-multi-shadow/reports
--control /var/lib/chartink-paper/mcx-paper/control.json
```

Observed listener:

```text
127.0.0.1:8765
PID 812870
```

Cloudflare process:

```text
/usr/bin/cloudflared tunnel
--url http://127.0.0.1:8765
--logfile /var/log/cloudflared.log
```

Recovered current Quick Tunnel hostname:

```text
https://triumph-events-chair-problems.trycloudflare.com
```

Therefore the current request path is proven as:

```text
triumph-events-chair-problems.trycloudflare.com
  -> cloudflared
  -> 127.0.0.1:8765
  -> mcx_paper.control_server
  -> /var/lib/chartink-paper/nifty-multi-shadow/reports
```

Both research pages returned HTTP 200 locally:

```text
/live_latest.html  HTTP 200  size 4,418,294 bytes
/mcx_latest.html   HTTP 200  size   153,445 bytes
```

## 3. D01 — NIFTY paper research runtime

### Service state

`nifty-multi-shadow-live.service` was inactive at audit time because the scheduled run had completed successfully.

Last run evidence:

```text
exit status: 0/SUCCESS
completed_bar_cutoff: 2026-09-18T15:10:00+05:30
signals: 150
open_paper_trades: 0
closed_paper_trades: 63
trades_used: 63
dashboard: /var/lib/chartink-paper/nifty-multi-shadow/reports/live_latest.html
```

This is consistent with a timer-triggered daily paper engine rather than a continuously running service.

### Generated dashboard artifact

```text
path: /var/lib/chartink-paper/nifty-multi-shadow/reports/live_latest.html
mode: 0600
owner/group: chartink-paper:chartink-paper
size: 4,418,294
mtime: 2026-09-18 09:41:44 UTC
SHA-256:
d4149db0ad25f6df0a2cd7f72e2167c62cb82409ffe0e20311994c8a1922c0df
```

### Git checkout

Observed:

```text
checkout: /opt/chartink-paper/nifty-options-v2
origin: git@github.com:kriskingg/etf-invest-engine.git
branch: chatgpt/statistical-options-buying-basket-v1
deployed checkout HEAD:
97f51a865a71bd44ddadd2007f609b4c083556a0
```

The server could not execute `git ls-remote` because the runtime user's current SSH context does not have GitHub public-key access.

Independently verified from GitHub on 2026-09-19:

```text
current remote branch HEAD:
f9a8fe8a22173fcda902bcbc2f8fd3a4da4defa5
```

Therefore the checkout metadata is **not at the current remote branch head**.

### Working-tree state

The checkout is intentionally/non-trivially dirty. It contains modified/deleted historical top-level docs, modified deployment scripts, and untracked runtime/materialized paths including:

```text
.git_sha
analysis/
replay_data
multiple NIFTY research/audit documents
```

Do not run destructive Git cleanup on this checkout.

### Runtime source versus tracked NIFTY source

Compared:

```text
runtime:
/opt/chartink-paper/nifty-options-v2/analysis/nifty_multi_shadow

tracked source:
/opt/chartink-paper/nifty-options-v2/nifty-options-paper-research/analysis/nifty_multi_shadow
```

The full recursive comparison reported exactly one source difference:

```text
dashboard.py differs
```

Critical files verified byte-identical between runtime and tracked source:

```text
engine.py
live.py
statistical_basket.py
ratio_hedge.py
evidence_exporter.py
health_monitor.py
research_review.py
oci/run_live.sh
```

The differing dashboard hashes are:

```text
runtime dashboard.py SHA-256:
831d3303996806cad52b7937e5a0607dc701acd1bd5e5fbe400e69c64f353272

server tracked-source dashboard.py SHA-256:
3b941c2b0389fd64b1e4d7be7efd034c57077423a7bd0e722e55397b0370896d
```

GitHub facts:

```text
dashboard.py Git blob at deployed checkout commit 97f51a8:
7a4a959784662ded8aec18b1c50c70585d2faa94

dashboard.py Git blob at current branch head f9a8fe8:
a6757e3087e643b0ac75f41ce993e371b3aac35b
```

A final direct runtime blob check proved:

```text
runtime dashboard.py blob:
a6757e3087e643b0ac75f41ce993e371b3aac35b

current GitHub branch dashboard.py blob:
a6757e3087e643b0ac75f41ce993e371b3aac35b
```

Therefore the live runtime `dashboard.py` is exactly the current GitHub version from branch head `f9a8fe8...`.

GitHub comparison between deployed checkout HEAD `97f51a8...` and current branch head `f9a8fe8...` shows only two changed files: `dashboard.py` and `test_nifty_dashboard_v2.py`. The test file is outside the runtime source tree. Since the full recursive server comparison found only `dashboard.py` different from the old tracked runtime source, and that runtime file exactly matches current GitHub, the deployed NIFTY runtime source tree is effectively current for the executable NIFTY application code.

### D01 confidence

```text
RUNTIME_SOURCE_PROVEN_CURRENT
CHECKOUT_METADATA_STALE
```

Proven:
- server;
- ingress;
- serving process;
- generated HTML;
- generator service/path;
- branch identity;
- most critical runtime files byte-identical to deployed tracked source.

Deployment-layout drift:
- checkout HEAD remains `97f51a8...`, behind current GitHub branch head `f9a8fe8...`;
- the executable runtime tree is an untracked top-level `analysis/` copy;
- that runtime tree contains the current `dashboard.py` even though the tracked checkout copy remains old.

This is not evidence of stale executable NIFTY source; it is evidence of a hybrid deployment layout whose Git metadata does not describe the actual runtime tree cleanly. Do not run destructive Git cleanup.

## 4. D02 — MCX / Silver hedge runtime

### Active generator

```text
hedge-engine-paper.service
status: active (running)
MainPID: 992037
working directory: /opt/hedge-engine
module: hedge_engine.runtime.paper_loop
poll interval: 5.0 seconds
```

Publication target:

```text
/var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
```

Primary report:

```text
/var/lib/hedge-engine/reports/mcx_latest.html
```

### Git checkout

Observed:

```text
origin: git@github.com:kriskingg/hedge-engine.git
branch: main
deployed HEAD:
e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071
```

Fresh GitHub verification on 2026-09-19:

```text
remote main HEAD:
e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071
```

Therefore the deployed MCX source commit is exactly the current GitHub `main` commit.

The server-side `git ls-remote` command failed only because the runtime shell lacks the GitHub SSH key; GitHub was independently checked through the connected repository.

### Full tracked-file byte check

The audit iterated every Git-tracked file and compared:

```text
git hash-object <working file>
vs
git rev-parse HEAD:<path>
```

No tracked-file differences were printed.

Therefore all Git-tracked files in `/opt/hedge-engine` are byte-identical to deployed HEAD `e10eeeda...`.

Untracked files observed:

```text
deploy_bundle.tar.gz
deploy_bundle_70ddf70.tar.gz
```

These deployment archives are not source drift, but they should remain documented as untracked runtime artifacts.

### Critical source Git blob proof

```text
paper_loop.py
80a08dcd3fdf4ba07e67ca8ac4ed3e87cfad8b2b

renderer.py
faf62c3ad8c26aa6bbdc6f9aed01a44bfa316d66

renderer_core.py
e2d997e7a5f8300d5fce0d756f024dc02afe481b

charts.py
758b5ce07c466e7fd597c3c9e29464302749fa7e

next_action.py
fa51c7c98c09694226509f8aeeb927d1916e568f

position_control.py
ce5863ce4d50e836251f84c4f4232498bb6641f7

theme_colorful.py
5499fe12ef70d3ed86df184521e5ec7a92d422fe

deploy/systemd/hedge-engine-paper.service
97052b7d1d50e1b84065ecd56c503b4eedceef26
```

Each critical working-tree blob matched its deployed HEAD blob.

### Generated MCX artifact proof

At the final comparison:

```text
primary report:
/var/lib/hedge-engine/reports/mcx_latest.html

published report:
/var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html

SHA-256 for both:
b7c603ea602ad2fd4932df1d53a82f1c03f33aefbcd3931019e289cfa0d9f3fc

MCX_HTML_COMPARE=IDENTICAL
```

The report changes continuously because `paper_loop` refreshes it every five seconds. Earlier in the same audit a different SHA-256 was observed for the published page; this is expected for a live regenerated artifact. The important proof is that primary and published copies were identical when compared together.

### Legacy writer check

`mcx-paper-live.service` exists but was:

```text
disabled
inactive (dead)
```

The active MCX generator observed during this audit is `hedge-engine-paper.service`.

The shared `mcx_paper.control_server` is the HTTP/control serving layer, not the current `mcx_latest.html` generator.

### Installed systemd unit drift

Repository unit:

```text
/opt/hedge-engine/deploy/systemd/hedge-engine-paper.service
SHA-256:
3d886c4fd79856f25a23719f2c8eed4746e88af15877df3711072b2e2e5c1852
```

Installed unit:

```text
/etc/systemd/system/hedge-engine-paper.service
SHA-256:
5a659e331364f5d49c7d34ee089f3e3e7595d20e633d9dfadd5da180c135f66c
```

Result:

```text
SYSTEMD_UNIT_COMPARE=DIFFERENT
```

The final direct diff proved the content is semantically identical and differs only by line endings:

```text
repository file: LF
installed unit:  CRLF
```

The first byte difference occurs where the repository has LF (`0x0A`) and the installed unit has CRLF (`0x0D 0x0A`). The unified diff shows every logical line unchanged.

Therefore the systemd discrepancy is **non-semantic line-ending drift only**.

### D02 confidence

Source/runtime code:

```text
PROVEN_BYTE_IDENTICAL
```

Installed service definition:

```text
SEMANTICALLY_IDENTICAL_CRLF_ONLY
```

Overall D02:

```text
PROVEN_CURRENT_RUNTIME
```

The application source is exact and current. The installed systemd unit differs only in LF versus CRLF line endings, with no logical configuration difference.

## 5. Cross-dashboard conclusion

Fresh runtime evidence confirms:

```text
chartink-paper / 10.0.0.71
documented public IP 80.225.234.65
        |
        +-- cloudflared -> 127.0.0.1:8765
        |
        +-- mcx_paper.control_server
              |
              +-- live_latest.html
              |     generated by NIFTY multi-shadow runtime
              |
              +-- mcx_latest.html
                    generated by hedge-engine-paper.service
```

The exact current Cloudflare hostname is:

```text
https://triumph-events-chair-problems.trycloudflare.com
```

No evidence from this audit supports treating the shared HTTP/control server as the generator owner of both applications.

## 6. Dashboard ownership closure

All requested `chartink-paper` provenance questions are now resolved.

D01:
- executable NIFTY runtime source is proven current for the application runtime tree;
- checkout metadata remains stale/hybrid at `97f51a8...`, so deployment hygiene is imperfect but runtime code ownership is established.

D02:
- tracked hedge-engine runtime source is byte-identical to current GitHub `main`;
- generated primary and published MCX pages matched byte-for-byte;
- installed systemd unit differs only by CRLF versus LF line endings.

No further `chartink-paper` verification is required for dashboard ownership mapping.

## 7. Cloudflare reboot and hostname durability

Fresh final-gap evidence proved:

```text
service: cloudflared-tunnel.service
enabled: yes
active: yes
Restart=always
ExecStart:
/usr/bin/cloudflared tunnel --url http://127.0.0.1:8765 --logfile /var/log/cloudflared.log
```

The current process started on 2026-08-17 and has remained running. Retained
logs showed only the current hostname:

```text
triumph-events-chair-problems.trycloudflare.com
```

This is a **Quick Tunnel**, not a named tunnel. Cloudflare's current
documentation states that Quick Tunnels generate a random
`*.trycloudflare.com` subdomain and are intended for testing/development, not
production. Therefore the current hostname is not a durable recovery address and
a fresh cloudflared process must be expected to obtain a new random hostname.

## 8. Public control-surface security

The shared server source at
`/opt/chartink-paper/mcx-paper-research/mcx_paper/control_server.py` has
SHA-256:

```text
597ed799d3267f93f69751f2072d8db0ec31627094bf1d6f0c78080538a607e0
```

Observed methods/path:

```text
GET  /api/mcx-paper-control
POST /api/mcx-paper-control
```

GET returns the control state. POST ultimately calls
`atomic_control(... enabled=action == "start")` and therefore mutates the
control file.

The POST checks a fixed header:

```text
X-Paper-Control: local-dashboard
```

and also inspects `Origin`. No secret/token/password/API-key authentication
identifier was detected by the source audit. A fixed client header and Origin
validation are not strong authentication controls for an Internet-facing
endpoint: non-browser clients can supply their own headers. Treat this as a
**security remediation blocker**.

The Quick Tunnel proxies the entire local HTTP origin. Public read-only tests
proved both dashboards reachable through the Internet-facing hostname:

```text
/live_latest.html  -> HTTP 200
/mcx_latest.html   -> HTTP 200
```

No mutating POST was executed during the audit.

Preferred target architecture for later review: keep public/static research
viewing separate from any mutating operator-control API, or protect the control
surface with a deliberate authenticated/private access boundary. No runtime
change was made by this audit.

## 9. Failed scheduled NIFTY units

Three failed units are not equivalent.

### nifty-multi-shadow.service — active operational defect

Its timer is enabled and schedules 15:50 IST weekdays. The service failed on
Sep 15, 16, 17 and 18 at:

```text
sqlite3.OperationalError: unable to open database file
```

The current GitHub `run_daily.sh` selects:

```text
/var/lib/chartink-paper/nifty-options-v2/data/research-YYYY-MM-DD.sqlite3
```

and the quality code opens it read-only before `PRAGMA integrity_check`.
Because the failure repeats on an enabled scheduled path, classify it as an
**active operational defect** until the physical runtime path/permission/WAL
cause is proven and fixed.

### nifty-options-v2-replay.service — broken scheduled path / retirement candidate

The service itself is disabled, but
`nifty-options-v2-replay.timer` is **enabled** and triggers it every weekday at
15:40 IST. Sep 15-18 all failed with:

```text
status=203/EXEC
```

The timer therefore makes the disabled service relevant. The follow-up audit
also proved the target script exists with a valid `#!/usr/bin/env bash`
shebang but mode `0664` (no executable bit), while the working multi-shadow
script is mode `0755`. This directly explains systemd `203/EXEC`.

Classify the immediate defect as **deployed executable permission missing**.
Still decide separately whether this older replay path should be repaired or
retired; do not simply chmod it as part of the architecture audit.

### nifty-multi-shadow-evidence-exporter.service — evidence-delivery defect

The exporter successfully verified remote publication for many older COMPLETE
sessions. Historical INCOMPLETE sessions with manifests are correctly left
unpublished. However the Sep 18 run recorded eight operational failures for
Sep 8-18 sessions where the exporter reported INCOMPLETE / zero files and then
produced no manifest.

Therefore:

```text
older COMPLETE evidence: remote publication verified
recent evidence delivery: unhealthy / incomplete
service result: failed
timer: enabled, multiple weekday retries
```

This is a research-evidence delivery problem, not a trading-engine tuning
problem.

## 10. Mutable MCX state and boot-volume durability

Fresh inventory under `/var/lib/hedge-engine` measured about 26 MiB and
included:

```text
data/mcx_scrip_master.csv              ~25.7 MB
data/dashboard_price_history.jsonl     ~867 KB
executions/execution_history.jsonl
state/runtime_checkpoint.json
state/etf_ledger.json
state/strategy_state.json
state/runtime_health.json
reports/mcx_latest.html
backups/backup_*/
```

The live `hedge_engine.runtime.paper_loop` updates state/report files and the
published MCX dashboard.

`findmnt` proved `/var/lib/hedge-engine`, `/opt/hedge-engine`,
`/var/lib/chartink-paper` and `/opt/chartink-paper` all resolve to the VM
root filesystem on `/dev/sda1` (ext4). Thus this mutable state survives an
ordinary reboot if the boot volume survives, but is not independently
VM-loss-safe without boot-volume or off-server backup.

Local pre-deploy/runtime backups under `/var/lib/hedge-engine/backups` are on
the same filesystem and do not create a separate failure domain.

## 11. Remaining durability unknowns

Two inputs remain before declaring the chartink-paper durability audit fully
closed:

1. The follow-up audit proved `/var/lib/chartink-paper` is mode `0700`,
   owned by `chartink-paper`. Because the audit command ran as `ubuntu`, it
   could not traverse that tree. The previous 4 KiB result was therefore a
   permissions artifact, not proof of symlinks. If raw NIFTY durability must be
   fully closed, rerun only the inventory portion as root or the
   `chartink-paper` service user.
2. OCI boot-volume backup policy and existing boot-volume backups remain
   **UNKNOWN** from the host because OCI CLI is not installed. Verify from OCI
   control-plane evidence rather than installing tooling solely for the audit.

Dashboard/code ownership is closed; security/durability cleanup is not.
