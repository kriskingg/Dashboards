# D02 — MCX / Silver Hedge Research Dashboard

**Status:** PROVEN CURRENT RUNTIME; SYSTEMD COPY DIFFERS ONLY BY LINE ENDINGS  
**GitHub verification:** 2026-09-19  
**Fresh OCI verification:** 2026-09-19  
**Browser URL:** `https://triumph-events-chair-problems.trycloudflare.com/mcx_latest.html`

## Purpose

This is the MCX / Silver hedge paper-research dashboard. It is a separate research system from the NIFTY options paper dashboard even though both pages currently share the same Cloudflare Quick Tunnel, local HTTP/control service, port, and report directory.

The source repository is:

```text
kriskingg/hedge-engine
branch: main
```

The deployed OCI checkout and current GitHub `main` are both:

```text
e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071
```

## End-to-end architecture

Fresh server evidence proves:

```text
Browser
  |
  | HTTPS
  v
Cloudflare Quick Tunnel
triumph-events-chair-problems.trycloudflare.com
  |
  | -> http://127.0.0.1:8765
  v
chartink-paper
private IP observed: 10.0.0.71
documented public IP: 80.225.234.65
  |
  v
nifty-multi-shadow-dashboard.service
  |
  v
mcx_paper.control_server
  |
  | web root:
  | /var/lib/chartink-paper/nifty-multi-shadow/reports
  v
mcx_latest.html
  ^
  |
  | published by:
  |
hedge-engine-paper.service
  |
  v
/opt/hedge-engine
python -m hedge_engine.runtime.paper_loop
  |
  +--> src/hedge_engine/dashboard/renderer.py
  +--> src/hedge_engine/dashboard/renderer_core.py
  |
  v
primary report:
/var/lib/hedge-engine/reports/mcx_latest.html
```

## Active runtime proof

Observed service:

```text
hedge-engine-paper.service
Active: active (running)
MainPID: 992037
WorkingDirectory=/opt/hedge-engine
```

Observed command:

```text
/opt/hedge-engine/.venv/bin/python
-m hedge_engine.runtime.paper_loop
--state-dir /var/lib/hedge-engine/state
--data-dir /var/lib/hedge-engine/data
--reports-dir /var/lib/hedge-engine/reports
--executions-dir /var/lib/hedge-engine/executions
--publish-path /var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
--poll-interval 5.0
```

## Git/runtime proof

Observed OCI checkout:

```text
origin: git@github.com:kriskingg/hedge-engine.git
branch: main
HEAD: e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071
```

Fresh GitHub verification found `main` at the same SHA.

The server audit compared every tracked file in the working tree using:

```text
git hash-object <file>
vs
git rev-parse HEAD:<file>
```

No tracked-file mismatch was reported.

Therefore:

```text
/opt/hedge-engine tracked source
= deployed HEAD e10eeeda...
= current GitHub main e10eeeda...
```

for all Git-tracked files at audit time.

Untracked artifacts:

```text
deploy_bundle.tar.gz
deploy_bundle_70ddf70.tar.gz
```

These are deployment artifacts and are not part of the tracked source proof.

## Critical source blob proof

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

## Generated artifact proof

The paper engine maintains:

```text
primary:
/var/lib/hedge-engine/reports/mcx_latest.html

published:
/var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
```

At the final comparison both had:

```text
SHA-256:
b7c603ea602ad2fd4932df1d53a82f1c03f33aefbcd3931019e289cfa0d9f3fc

MCX_HTML_COMPARE=IDENTICAL
```

The file is live and continuously regenerated, so its hash can change between observations. The proof is that primary and published copies were byte-identical when compared together.

## Legacy writer check

`mcx-paper-live.service` exists but was:

```text
disabled
inactive (dead)
```

The active writer observed during the audit is `hedge-engine-paper.service`.

The shared `mcx_paper.control_server` serves the file and exposes paper control; it is not the current generator owner of `mcx_latest.html`.

## Systemd unit drift

This is the only unresolved D02 byte-level discrepancy.

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

The final direct diff proved the logical content is identical. The repository copy uses LF line endings, while the installed unit uses CRLF line endings. The first byte difference is exactly LF versus CRLF; no configuration line differs semantically.

Therefore this is a **non-semantic line-ending difference only**, not operational service drift.

## Shared-ingress boundary

```text
live_latest.html -> NIFTY generator: kriskingg/etf-invest-engine
mcx_latest.html  -> MCX generator: kriskingg/hedge-engine
```

Do not infer generator ownership from the shared hostname, shared port 8765, shared web root, or the fact that the serving module is named `mcx_paper.control_server`.

## Confidence

Application source:

```text
PROVEN_BYTE_IDENTICAL
```

Overall D02:

```text
PROVEN_CURRENT_RUNTIME
```

See [CHARTINK_PAPER_RUNTIME_VERIFICATION_2026-09-19.md](CHARTINK_PAPER_RUNTIME_VERIFICATION_2026-09-19.md) for the complete byte-level audit record.
## Code-change and repository decision guide

For exact files/tests to modify for D02, plus repository migration/retention decisions, see [Dashboard Code Ownership, Change Guide and Repository Boundaries](DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md) and [Dashboard Repository Boundary Decision Register](DASHBOARD_REPOSITORY_BOUNDARY_DECISIONS.md).
