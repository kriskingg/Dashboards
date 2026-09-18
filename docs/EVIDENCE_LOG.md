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
