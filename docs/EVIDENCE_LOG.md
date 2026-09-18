# Evidence Log

This file records the concrete evidence used to prove dashboard ownership. It is intentionally factual and dated.

## 2026-09-18 — D01 NIFTY Paper Research

### Browser evidence

Observed page:

```text
https://triumph-events-chair-problems.trycloudflare.com/live_latest.html
```

Visible identifiers included:

```text
NIFTY paper research
Live forward-testing dashboard
PAPER ONLY
MCX COMMODITIES
Evidence cleanup
retained trades
```

### Generated file evidence

Server:

```text
chartink-paper
```

File:

```text
/var/lib/chartink-paper/nifty-multi-shadow/reports/live_latest.html
```

Observed metadata:

```text
Size: 4418294 bytes
Owner: chartink-paper:chartink-paper
Mode: 0600
Modify: 2026-09-18 09:41:44 UTC
```

The HTML was observed to be a large self-contained generated artifact embedding trade/signal data and UI JavaScript.

### Source-string tracing

Searching server source trees for distinctive generated-dashboard identifiers returned:

```text
/opt/chartink-paper/nifty-options-v2/analysis/nifty_multi_shadow/dashboard.py
/opt/chartink-paper/nifty-options-v2/analysis/nifty_multi_shadow/live.py
```

Matching identifiers included:

```text
renderOverviewV2
DASHBOARD_DATA
NIFTY paper research
Evidence cleanup
live_latest.html
```

### Runtime Git evidence

Runtime tree:

```text
/opt/chartink-paper/nifty-options-v2
```

Remote:

```text
origin git@github.com:kriskingg/etf-invest-engine.git
```

Active branch:

```text
chatgpt/statistical-options-buying-basket-v1
```

HEAD:

```text
97f51a865a71bd44ddadd2007f609b4c083556a0
```

Commit subject:

```text
Test evidence delivery end-to-end acceptance checks
```

### Source blob proof

Runtime dashboard file:

```text
analysis/nifty_multi_shadow/dashboard.py
a6757e3087e643b0ac75f41ce993e371b3aac35b
```

GitHub authoritative dashboard source:

```text
nifty-options-paper-research/analysis/nifty_multi_shadow/dashboard.py
a6757e3087e643b0ac75f41ce993e371b3aac35b
```

Runtime live file:

```text
analysis/nifty_multi_shadow/live.py
4a4db2288563374e4fed4e9d8e6d6a3cb7ed138a
```

GitHub authoritative live source:

```text
nifty-options-paper-research/analysis/nifty_multi_shadow/live.py
4a4db2288563374e4fed4e9d8e6d6a3cb7ed138a
```

Conclusion: both runtime source files matched GitHub byte-for-byte.

### Runtime service evidence

Unit:

```text
nifty-multi-shadow-live.service
```

Relevant configuration:

```text
User=chartink-paper
Group=chartink-paper
WorkingDirectory=/opt/chartink-paper/nifty-options-v2
ExecStart=/opt/chartink-paper/nifty-options-v2/analysis/nifty_multi_shadow/oci/run_live.sh
ReadOnlyPaths=/opt/chartink-paper/nifty-options-v2 /var/lib/chartink-paper/nifty-options-v2
ReadWritePaths=/var/lib/chartink-paper/nifty-multi-shadow
```

### Serving evidence

HTTP service:

```text
nifty-multi-shadow-dashboard.service
```

Listening endpoint:

```text
127.0.0.1:8765
```

Current implementation:

```text
/opt/chartink-paper/mcx-paper-research/.venv/bin/python
-m mcx_paper.control_server
--bind 127.0.0.1
--port 8765
--directory /var/lib/chartink-paper/nifty-multi-shadow/reports
```

Local HTTP response for the dashboard returned `200 OK`.

### Cloudflare evidence

Running quick-tunnel command:

```text
/usr/bin/cloudflared tunnel --url http://127.0.0.1:8765 --logfile /var/log/cloudflared.log
```

The active process held a rotated logfile and the existing hostname was recovered from its open file descriptor:

```text
https://triumph-events-chair-problems.trycloudflare.com
```

### Dirty checkout evidence

The runtime checkout was not clean. It contained many modified/deleted documentation paths, modified deployment files, and untracked paths including:

```text
analysis/
.git_sha
replay_data
multiple research/audit documents
```

This is why the D01 record explicitly prohibits blind destructive Git cleanup.

### Final evidence conclusion

```text
D01 ownership: PROVEN
Generator owner: kriskingg/etf-invest-engine
Active branch: chatgpt/statistical-options-buying-basket-v1
Runtime: /opt/chartink-paper/nifty-options-v2
Serving layer: shared mcx_paper.control_server
Ingress: Cloudflare quick tunnel
```


---

## 2026-09-19 — D02 MCX / Silver Hedge Source Ownership

### Scope of this review

This review used GitHub source plus the serving facts already recorded for D01. It did **not** directly inspect the OCI filesystem or systemd state on 2026-09-19.

Therefore this entry proves source ownership and deployment intent, but does not claim that the current OCI checkout is byte-for-byte identical to the latest GitHub `main` branch.

### Browser identity already recorded

MCX research page:

```text
https://triumph-events-chair-problems.trycloudflare.com/mcx_latest.html
```

The existing dashboard registry records that this page shares the same quick tunnel, port 8765 serving layer, and report directory as D01.

### Repository discovery

Relevant repositories under `kriskingg` were inspected. The dedicated implementation repository is:

```text
kriskingg/hedge-engine
default branch: main
```

The latest default-branch commit observed during this review was:

```text
e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071
Merge pull request #3 from kriskingg/feat/position-control-modal-basket
```

### Deployment unit evidence

File:

```text
kriskingg/hedge-engine
deploy/systemd/hedge-engine-paper.service
```

The service contract runs:

```text
/opt/hedge-engine/.venv/bin/python -m hedge_engine.runtime.paper_loop
```

with:

```text
WorkingDirectory=/opt/hedge-engine
PYTHONPATH=/opt/hedge-engine/src
```

and publishes:

```text
--publish-path /var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
```

This is direct source evidence that hedge-engine owns generation/publication of the MCX page.

### Runtime source evidence

`src/hedge_engine/runtime/paper_loop.py` defines:

```text
reports_dir = /var/lib/hedge-engine/reports
published_report_path = /var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
primary_report_path = /var/lib/hedge-engine/reports/mcx_latest.html
```

GitHub blob at review:

```text
paper_loop.py
80a08dcd3fdf4ba07e67ca8ac4ed3e87cfad8b2b
```

### Dashboard renderer evidence

Primary composition module:

```text
src/hedge_engine/dashboard/renderer.py
```

It composes the established dashboard from:

```text
src/hedge_engine/dashboard/renderer_core.py
```

and adds the next-action and position-control panels.

Relevant GitHub blobs:

```text
renderer.py
faf62c3ad8c26aa6bbdc6f9aed01a44bfa316d66

renderer_core.py
e2d997e7a5f8300d5fce0d756f024dc02afe481b
```

### Migration/runbook evidence

`docs/MCX_RUNTIME_MIGRATION.md` explicitly records the target architecture:

```text
Code:  /opt/hedge-engine
State: /var/lib/hedge-engine
```

It also explicitly identifies:

```text
https://triumph-events-chair-problems.trycloudflare.com/mcx_latest.html
```

as the MCX presentation URL and says the NIFTY options `/live_latest.html` runtime is a separate boundary.

### Repository boundary evidence

The `kriskingg/hedge-engine` README states that `kriskingg/etf-invest-engine` is read-only reference material for this project and that implementation changes belong in `kriskingg/hedge-engine`.

This resolves the generator ownership question independently from the shared report path.

### Cross-dashboard conclusion

```text
Shared ingress:
triumph-events-chair-problems.trycloudflare.com

Shared serving target:
127.0.0.1:8765

Shared report directory:
/var/lib/chartink-paper/nifty-multi-shadow/reports

NIFTY page:
live_latest.html
Generator owner: kriskingg/etf-invest-engine

MCX page:
mcx_latest.html
Generator owner: kriskingg/hedge-engine
```

### Remaining runtime proof required

To promote D02 from source-proven to fully runtime-proven, obtain a fresh OCI check of:

```text
/opt/hedge-engine git remote
deployed branch
deployed HEAD
systemctl cat hedge-engine-paper.service
hashes of deployed paper_loop.py / renderer.py / renderer_core.py
mcx_latest.html freshness
service active state
```

### Final evidence conclusion

```text
D02 source ownership: PROVEN
Generator owner: kriskingg/hedge-engine
Source branch: main
Runtime deployment SHA: NOT RE-VERIFIED IN THIS REVIEW
Serving layer: shared mcx_paper.control_server
Ingress: shared Cloudflare quick tunnel
```
