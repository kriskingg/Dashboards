# All Dashboards Session Turnover / Continuation Guide — 2026-09-19

## Read this first

This is the **cross-dashboard handoff** for future ChatGPT sessions.

There are currently **6 confirmed dashboards: D01-D06**. Do not assume every dashboard lives in the same repository, on the same server, or has the same safety boundary.

The central ownership/documentation repository is:

```text
kriskingg/Dashboards
```

Repository:
https://github.com/kriskingg/Dashboards

Canonical dashboard index:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/README.md

Master system map:
https://github.com/kriskingg/Dashboards/blob/main/docs/MASTER_SYSTEM_MAP.md

Code-change/ownership guide:
https://github.com/kriskingg/Dashboards/blob/main/docs/DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md

Repository-boundary decisions:
https://github.com/kriskingg/Dashboards/blob/main/docs/DASHBOARD_REPOSITORY_BOUNDARY_DECISIONS.md

Infrastructure audit checklist:
https://github.com/kriskingg/Dashboards/blob/main/docs/INFRASTRUCTURE_AUDIT_CHECKLIST.md

Important rule:

> GitHub branch heads and deployed runtime SHAs are different facts. Always verify both before changing a dashboard. Do not overwrite a last-observed deployed SHA merely because the GitHub branch advanced.

---

## Current confirmed dashboard inventory

| ID | Dashboard | Source repository | Main source/ref | Runtime location | Safety boundary |
|---|---|---|---|---|---|
| D01 | NIFTY Paper Research | `kriskingg/etf-invest-engine` | `chatgpt/statistical-options-buying-basket-v1` | `chartink-paper:/opt/chartink-paper/nifty-options-v2` | PAPER/SHADOW only |
| D02 | MCX / Silver Hedge Research | `kriskingg/hedge-engine` | `main` | `chartink-paper:/opt/hedge-engine` | paper-research / hedge runtime |
| D03 | ETF / Kotak Operator Dashboard | `kriskingg/etf-invest-engine` | intended production authority `main`; deployed branch last observed separately | `lakshmidevi:/home/ubuntu/kotak` | LIVE broker/operator surface |
| D04 | ETF Platform-v2 | `kriskingg/etf-invest-engine` | same ETF domain; feature/deployed branch last observed separately | same `lakshmidevi` Uvicorn process | LIVE broker/domain backend, separate frontend |
| D05 | Mutual Funds Analytics / Tactical Research | `kriskingg/mf-analytics-source` | `main` + preservation/path-cleanup branches pending integration | local Windows `D:\Git_repos\mf-analytics-source` | local research/analytics |
| D06 | Personal Investment Tracker | `kriskingg/investment-tracker-app` | `main` | local Windows `D:\Git_repos\investment-tracker-app` | local personal-finance app |

There are **6 dashboards**, not 6 independent strategy engines. D03 and D04 share one live ETF/Kotak backend/process; D01 and D02 share a serving layer but have independent generator/source ownership.

---

## Current GitHub ref snapshot

Verified from GitHub on 2026-09-19 during this turnover:

| System | Repository/ref | Current GitHub head |
|---|---|---|
| D01 | `kriskingg/etf-invest-engine` / `chatgpt/statistical-options-buying-basket-v1` | `6a37957bf29f45c99b79e537f4698fbcdcca3f66` |
| D02 | `kriskingg/hedge-engine` / `main` | `e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071` |
| D03 intended production | `kriskingg/etf-invest-engine` / `main` | `e52aa2ab7f8d4a260027778982b6abfe69203225` |
| D03/D04 feature branch | `kriskingg/etf-invest-engine` / `platform-v2-v03-preimplementation-20260916` | `b2def17c7746656ede1a9514551fe276a21e7c40` |
| D05 main | `kriskingg/mf-analytics-source` / `main` | `5366b9000d0f93af8e4e635b4fc551f0a85d0915` |
| D05 preserved local source | `preserve/d05-local-source-20260919` | `478e7a018ba0d587915445d4f7decfb056c63a0d` |
| D05 path cleanup | `chatgpt/d05-path-cleanup-20260919` | `a7bd4c0903867f6b5e4d378a502f27480c3813fe` |
| D06 | `kriskingg/investment-tracker-app` / `main` | `8f3a24bc7f30ae10d6695a6a292447351a7db5e9` |
| Registry | `kriskingg/Dashboards` / `main` before this turnover update | `8427703cdc08d8a2ab2a4c8de851974acb5471a1` |

These are **GitHub source refs**, not proof of what is currently deployed on either OCI server.

---

# D01 — NIFTY Paper Research

Canonical page:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D01-NIFTY-PAPER-RESEARCH.md

Deep evidence:
https://github.com/kriskingg/Dashboards/blob/main/docs/D01-nifty-paper-research.md

Runtime verification:
https://github.com/kriskingg/Dashboards/blob/main/docs/CHARTINK_PAPER_RUNTIME_VERIFICATION_2026-09-19.md

Source:
https://github.com/kriskingg/etf-invest-engine/tree/chatgpt/statistical-options-buying-basket-v1/nifty-options-paper-research

Current GitHub branch head at turnover:
`6a37957bf29f45c99b79e537f4698fbcdcca3f66`.

Last audited runtime facts remain separate:
- host: `chartink-paper`;
- runtime checkout: `/opt/chartink-paper/nifty-options-v2`;
- generated page: `/var/lib/chartink-paper/nifty-multi-shadow/reports/live_latest.html`;
- shared local serving origin: `127.0.0.1:8765`;
- public access through a Cloudflare Quick Tunnel;
- deployed checkout metadata was hybrid/stale during the audit even though the relevant runtime dashboard source had been proven current at that time.

Do not infer that the server automatically moved to the newer GitHub branch head. Re-verify deployed SHA/file hashes before deployment or code review that depends on runtime parity.

Pending D01 work:
- continue NIFTY repository-ownership cleanup/extraction planning without changing strategy behavior;
- normalize the hybrid deployment safely;
- repair/close the known replay/evidence-export operational defects separately from strategy changes;
- complete durability/backup evidence for raw NIFTY state;
- preserve all frozen PAPER/SHADOW strategy/risk/causality invariants;
- never use a dashboard/code migration as an excuse to tune trading logic.

---

# D02 — MCX / Silver Hedge Research

Canonical page:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D02-MCX-SILVER-HEDGE.md

Deep evidence:
https://github.com/kriskingg/Dashboards/blob/main/docs/D02-mcx-silver-hedge.md

Source:
https://github.com/kriskingg/hedge-engine

Current GitHub `main` head:
`e10eeeda6ea17a7a6c5bc62ad0877b01cf8ef071`.

Last audited runtime:
- host: `chartink-paper`;
- checkout: `/opt/hedge-engine`;
- service: `hedge-engine-paper.service`;
- state root: `/var/lib/hedge-engine`;
- generated report: `/var/lib/hedge-engine/reports/mcx_latest.html`;
- published copy is served beside D01.

Pending D02 work:
- keep D02 source in `hedge-engine`;
- separate the shared public serving layer from mutating MCX paper control;
- do not broaden the current public control surface;
- complete durable backup/restore policy for mutable hedge state;
- keep strategy/domain changes separate from serving/UX work.

---

# D03 — ETF / Kotak Live Operator Dashboard

Canonical page:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D03-ETF-KOTAK-OPERATOR.md

Deep evidence:
https://github.com/kriskingg/Dashboards/blob/main/docs/D03-etf-kotak-private-dashboard.md

Runtime verification:
https://github.com/kriskingg/Dashboards/blob/main/docs/LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md

Source repository:
https://github.com/kriskingg/etf-invest-engine

Current GitHub refs at turnover:
- `main`: `e52aa2ab7f8d4a260027778982b6abfe69203225`;
- `platform-v2-v03-preimplementation-20260916`: `b2def17c7746656ede1a9514551fe276a21e7c40`.

Last observed deployed runtime on 2026-09-19:
- host: `lakshmidevi`;
- checkout: `/home/ubuntu/kotak`;
- deployed branch observed: `platform-v2-v03-preimplementation-20260916`;
- deployed HEAD observed at that audit: `306bd5c20dc48c682dab8c85ce1b2c677f97f66a`;
- Uvicorn origin: `127.0.0.1:8080`;
- Tailscale URL: `https://etf-trader.tailabfd53.ts.net/`.

The feature branch has advanced on GitHub since the last observed deployed SHA. Therefore **GitHub branch head != proven deployed head** until the server is re-audited.

Pending D03 work:
- reconcile intended production authority `main` vs the deployed feature branch;
- review the weekday 08:57 IST fetch/reset deployment path before it can redefine live code;
- establish deliberate reboot-persistent startup for the dashboard origin;
- define/test off-VM backup/restore for local operational state;
- never use a live broker order as a deployment test;
- preserve all broker/order/safety gates when changing UI or backend code.

---

# D04 — ETF Platform-v2

Canonical page:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D04-ETF-PLATFORM-V2.md

Deep evidence:
https://github.com/kriskingg/Dashboards/blob/main/docs/D04-etf-dashboard-v2.md

Source repository:
https://github.com/kriskingg/etf-invest-engine

D04 is a distinct React/TypeScript/Vite frontend but shares the D03 FastAPI/Uvicorn process and ETF domain backend.

Current GitHub feature-branch head:
`b2def17c7746656ede1a9514551fe276a21e7c40`.

Last audited deployed branch/checkout evidence is the same D03 server audit and must not be assumed to have advanced automatically.

Pending D04 work:
- reconcile branch/deployment ownership with D03;
- establish the final `/api/v2` contract;
- compare D03 vs D04 feature coverage and operator purpose before retiring either;
- make frontend build/deployment deterministic;
- keep broker truth and order safety server-side;
- only consider a frontend-only repository split after the API and production branch are stable.

---

# D05 — Mutual Funds Analytics / Tactical Research

Canonical page:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D05-MUTUAL-FUNDS-ANALYTICS.md

Dedicated D05 turnover:
https://github.com/kriskingg/Dashboards/blob/main/docs/D05_SESSION_TURNOVER_2026-09-19.md

Migration verification:
https://github.com/kriskingg/Dashboards/blob/main/docs/D05_LOCAL_MIGRATION_VERIFICATION_2026-09-19.md

Source repository:
https://github.com/kriskingg/mf-analytics-source

Critical repository guard:
**D05 is not in `etf-invest-engine`.**

Canonical local workspace:
`D:\Git_repos\mf-analytics-source`

Current GitHub refs:
- `main`: `5366b9000d0f93af8e4e635b4fc551f0a85d0915`;
- preserved local source: `478e7a018ba0d587915445d4f7decfb056c63a0d`;
- path-cleanup branch: `a7bd4c0903867f6b5e4d378a502f27480c3813fe`.

Open path-cleanup PR:
https://github.com/kriskingg/mf-analytics-source/pull/1

Pending D05 work:
- verify the local working tree is still clean;
- combine preservation + path-cleanup on an integration branch;
- deliberately resolve overlaps;
- verify zero tracked references to `D:\Dhan\Mutual_funds`;
- run backend tests;
- run frontend build/tests;
- runtime smoke-test on ports 8000/5173;
- create/review combined PR;
- merge only after code/test/runtime review;
- update Dashboards with final merged SHA;
- keep migration backups until final acceptance.

Do not merge PR #1 independently before the combined branch is validated.

---

# D06 — Personal Investment Tracker

Canonical page:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D06-INVESTMENT-TRACKER.md

Source:
https://github.com/kriskingg/investment-tracker-app

Current GitHub `main` head:
`8f3a24bc7f30ae10d6695a6a292447351a7db5e9`.

Canonical local path:
`D:\Git_repos\investment-tracker-app`

Current executable start contract documented:
- backend `127.0.0.1:8005`;
- frontend `127.0.0.1:5175`.

The README's older 8000/5173 references are not the executable authority.

Important migration warning:
During the wider local-repository consolidation, D06 was identified as having local unpublished work that must be preserved/audited separately. Do not apply D05 reset/cleanup assumptions to D06.

Pending D06 work:
- fresh local working-tree inventory;
- preserve any unpublished source to GitHub before cleanup;
- reconcile stale documentation with executable startup contract;
- run backend tests and frontend build;
- preserve transaction/FIFO/Decimal financial-truth contracts;
- verify localhost-only binding remains intentional.

---

# Shared infrastructure and cross-dashboard work

## chartink-paper shared serving layer

D01 and D02 share:
- Cloudflare Quick Tunnel;
- `127.0.0.1:8765`;
- report-serving directory;
- `mcx_paper.control_server`.

They do **not** share strategy/generator ownership.

The shared server currently mixes static/read-only serving with mutating MCX paper control. This remains a security/ownership cleanup item.

Target direction already documented:
- version-controlled read-only research gateway;
- no broker credentials;
- no strategy imports;
- no state mutation;
- health/read-only serving only;
- move/replace MCX control under a private, properly authenticated domain-owned boundary.

## lakshmidevi shared D03/D04 runtime

D03 and D04 share:
- `/home/ubuntu/kotak`;
- `127.0.0.1:8080`;
- one Uvicorn/FastAPI process;
- Tailscale ingress;
- ETF/Kotak broker/domain backend.

They differ in presentation architecture and must be compared before any route retirement.

## Data durability

The dashboard program is not complete merely because UI/source ownership is mapped.

Open durability work includes:
- NIFTY raw-state inventory with sufficient permissions;
- OCI boot-volume backup policy verification;
- actual restore-drill evidence;
- off-VM backup policy for D03/D04 local operational state;
- off-VM backup/restore policy for mutable D02 hedge state;
- local D05/D06 source preservation separate from local data.

---

# Change-control rules for future sessions

Before changing any dashboard:

1. identify D01-D06;
2. read this turnover plus the dashboard's canonical page;
3. inspect the owning GitHub repository, not a project-level default repository;
4. verify current GitHub branch head;
5. verify deployed/local runtime separately where relevant;
6. classify the request as presentation, data contract, strategy/domain logic, serving/ingress, deployment, or data durability;
7. edit authoritative source only;
8. never edit generated HTML/`dist` as source;
9. run dashboard-specific tests;
10. run domain/safety tests when data/control/order/strategy behavior is touched;
11. validate the runtime without unsafe live actions;
12. record new source/deployed SHAs back in `kriskingg/Dashboards`.

Do not:
- infer one dashboard's repository from another;
- assume GitHub head equals deployed head;
- use destructive `git reset --hard` or `git clean` on an unpreserved working tree;
- delete migration/data backups before explicit acceptance;
- merge a branch merely because GitHub says it is mergeable;
- combine infrastructure cleanup with strategy tuning;
- use real broker orders as deployment tests.

---

# Recommended work order from this handoff

The current highest-priority local consolidation task is D05 because its source is already preserved and the integration work is staged.

After D05:
1. audit/preserve D06 local unpublished source;
2. re-verify D03/D04 live deployed branch/SHA before touching live ETF dashboard code;
3. reconcile D03/D04 production/deployment authority and reboot startup;
4. continue D01 deployment/repository normalization without strategy changes;
5. address the D01/D02 shared public serving/control boundary;
6. complete cross-system backup/restore evidence.

This order is an operational sequencing recommendation, not a requirement to mix independent codebases into one change.

---

# Quick start for the next ChatGPT session

Use:

> Continue the dashboard consolidation from the all-dashboard turnover page. First identify which of D01-D06 we are working on, verify that dashboard's current GitHub ref and runtime/local state, then continue only within its ownership boundary. For D05, also read the dedicated D05 turnover page. Do not merge, delete, reset, deploy, or alter strategy/live-order behavior until the relevant preservation/review/test gates pass.

---

# Handoff status

- Confirmed dashboards: **6 (D01-D06)**.
- Central registry: `kriskingg/Dashboards`.
- Current GitHub source refs were re-verified during this turnover.
- OCI runtime observations remain timestamped audit evidence and must be re-verified before deployment-sensitive work.
- D05 integration is the immediate unfinished repository task.
- D06 local preservation remains pending.
- D01-D04 each have documented runtime/ownership/security/durability work still open.
