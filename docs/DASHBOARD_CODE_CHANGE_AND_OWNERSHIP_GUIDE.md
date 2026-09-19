# Dashboard Code Ownership, Change Guide and Repository Boundaries

**Last verified:** 2026-09-19  
**Purpose:** answer, before any change, **which code owns each dashboard, which
files must be changed, which tests protect it, what runtime is affected, and
whether the code belongs in its current repository**.

This document is the canonical engineering decision map for D01-D04. The
`Dashboards` repository is a **documentation / ownership / decision registry**.
It must not become a fifth runtime application repository.

## 1. One-page decision table

| ID | Dashboard | Domain owner | Current source repo / branch | Primary code to change | Repository decision |
|---|---|---|---|---|---|
| D01 | NIFTY paper research | NIFTY options paper/shadow research | `kriskingg/etf-invest-engine` / `chatgpt/statistical-options-buying-basket-v1` | `nifty-options-paper-research/analysis/nifty_multi_shadow/dashboard.py`, `live.py` | **MIGRATION CANDIDATE:** extract the complete NIFTY paper-research subtree into its own dedicated repo |
| D02 | MCX / Silver hedge paper research | Silver/MCX hedge research | `kriskingg/hedge-engine` / `main` | `src/hedge_engine/dashboard/*`, `src/hedge_engine/runtime/paper_loop.py` | **KEEP:** hedge-engine is the correct domain repo |
| D03 | Live ETF/Kotak operator dashboard `/` | live ETF/MTF production | `kriskingg/etf-invest-engine`; intended authority `main`, observed deployment branch differs | `src/dashboard/*` and guarded live broker APIs | **KEEP:** dashboard is tightly coupled to live ETF safety/broker domain |
| D04 | ETF Platform-v2 `/v2/` | future/live ETF operator platform | currently `etf-invest-engine` feature/deployed branch `platform-v2-v03-preimplementation-20260916` | `web/src/*`, `src/platform_v2/*`, `src/dashboard/spa.py` | **KEEP FOR NOW:** optional frontend split only after API contract + production branch are stable |
| G01 | Shared research HTTP gateway | shared D01/D02 presentation/ingress | runtime `/opt/chartink-paper/mcx-paper-research/mcx_paper/control_server.py`; Git authority not proven | static report serving + current MCX control API | **HIGH-PRIORITY EXTRACTION:** replace with a versioned, read-only shared gateway; remove mutating control from public origin |

## 2. Repository boundary rule

A dashboard should live with the domain that owns its business/safety truth
unless it is a genuinely independent presentation client with a stable API
contract.

That produces these boundaries:

```text
kriskingg/Dashboards
  documentation / ownership / migration decisions only
  NO runtime strategy or broker-control code

PROPOSED: kriskingg/nifty-options-paper-research
  NIFTY recorder/replay/paper engine
  D01 dashboard generator
  NIFTY tests
  systemd/deploy definitions
  evidence exporter
  research docs

kriskingg/hedge-engine
  MCX/Silver hedge strategy
  paper runtime
  D02 dashboard generator
  MCX state/recovery tests

kriskingg/etf-invest-engine
  live ETF/MTF production strategy
  broker integration and live safety
  D03 backend/UI
  D04 backend
  D04 frontend until its API boundary is mature

PROPOSED OPTIONAL: kriskingg/research-dashboard-gateway
  read-only serving of generated D01/D02 artifacts
  ingress/access-control configuration
  NO NIFTY strategy code
  NO MCX strategy code
  NO broker credentials
  NO mutating paper-control endpoint
```

Not every dashboard needs its own repository. Splitting tightly coupled safety
code merely to achieve one-repo-per-screen would increase risk.

---

# 3. D01 — NIFTY Paper Research Dashboard

## 3.1 Ownership

```text
URL:
  /live_latest.html

Host:
  chartink-paper

Runtime generator:
  /opt/chartink-paper/nifty-options-v2/analysis/nifty_multi_shadow/

Git authority:
  kriskingg/etf-invest-engine
  branch chatgpt/statistical-options-buying-basket-v1

Git-tracked source:
  nifty-options-paper-research/analysis/nifty_multi_shadow/
```

The deployed layout is currently hybrid: the executable top-level
`analysis/nifty_multi_shadow` copy is not a clean representation of the
checkout metadata. Do not use a destructive Git reset/clean as a migration
shortcut.

## 3.2 If you want to change D01, edit this

| Desired change | Primary source |
|---|---|
| HTML layout, cards, tables, chart JS, labels, filters, analytics presentation | `nifty-options-paper-research/analysis/nifty_multi_shadow/dashboard.py` |
| Which state/data is passed into the page; regeneration lifecycle | `nifty-options-paper-research/analysis/nifty_multi_shadow/live.py` |
| Browser refresh policy | `nifty-options-paper-research/analysis/nifty_multi_shadow/dashboard_refresh.py` |
| Strategy names/order/categories displayed | primarily `dashboard.py`; underlying strategy registry in engine modules |
| Actual signal/trade logic | **not a dashboard change**; use `engine.py`, `statistical_basket.py`, pattern/volume modules, etc. under the NIFTY strategy contract |
| Derived VOLUME_2X1_HEDGE presentation/logic | `ratio_hedge.py` plus dashboard presentation |
| Live paper launcher/runtime paths | `analysis/nifty_multi_shadow/oci/run_live.sh` and its systemd definitions |
| Generated output | never edit `live_latest.html` directly |

Core UI blob verified on 2026-09-19:

```text
dashboard.py
a6757e3087e643b0ac75f41ce993e371b3aac35b
```

## 3.3 Required tests for dashboard changes

Minimum dashboard-specific regression:

```text
nifty-options-paper-research/analysis/test_nifty_dashboard_v2.py
```

That test protects dashboard evidence filtering, chart/data contracts,
aggregation, strategy overlap, finite JSON output and expected page elements.

If a change touches actual strategy/state generation, also run the full NIFTY
test suite and the engine's legacy/statistical acceptance gates. UI work must
not silently alter signal formulas, risk rules, evidence classification or
legacy invariance.

## 3.4 D01 serving layer is not its owner

D01 is served by:

```text
mcx_paper.control_server
127.0.0.1:8765
Cloudflare Quick Tunnel
```

That serving code does not generate D01. A CSS/chart/filter change belongs in
`dashboard.py`, not `control_server.py`.

## 3.5 Repository recommendation

**Recommended target:** dedicated NIFTY paper-research repository.

Why:

- the subtree already has a distinct domain, deployment model, tests, docs,
  systemd units and evidence lifecycle;
- it is paper/shadow only and has a different safety boundary from live ETF/MTF;
- keeping it only on a non-main branch of `etf-invest-engine` makes ownership
  and deployment unnecessarily ambiguous;
- the current server already behaves as if it were an independent application.

### What should migrate together

Do not move only `dashboard.py`. Extract the coherent application:

```text
nifty-options-paper-research/
  analysis/
  deploy/
  docs/
  scripts/
  fixtures/config needed by tests
  tests/evidence contracts belonging to NIFTY research
```

Preserve the `paper-forward-evidence` publication contract or deliberately
move it with a compatibility plan.

### Migration blockers / risks

1. Current OCI deployment uses hard-coded paths under
   `/opt/chartink-paper/nifty-options-v2`.
2. Runtime source is currently a hybrid top-level `analysis/` copy.
3. Evidence publication depends on Git branch/repo semantics.
4. Several scheduled replay/evidence jobs are currently unhealthy and must not
   be accidentally "fixed" by an unrelated repo move.
5. Raw state/data under `/var/lib/chartink-paper` is private to the
   `chartink-paper` service user; migration must preserve ownership/modes.
6. Historical Git lineage and regression fixtures should be preserved.
7. No strategy formulas, frozen risk rules or execution semantics may change
   during a repository-only migration.

---

# 4. D02 — MCX / Silver Hedge Research Dashboard

## 4.1 Ownership

```text
Git repository:
  kriskingg/hedge-engine

Branch:
  main

Runtime:
  /opt/hedge-engine

Generator:
  python -m hedge_engine.runtime.paper_loop

Primary report:
  /var/lib/hedge-engine/reports/mcx_latest.html

Published copy:
  /var/lib/chartink-paper/nifty-multi-shadow/reports/mcx_latest.html
```

## 4.2 If you want to change D02, edit this

| Desired change | Primary source |
|---|---|
| Overall HTML rendering/composition | `src/hedge_engine/dashboard/renderer.py`, `renderer_core.py` |
| Price/P&L/position charts | `src/hedge_engine/dashboard/charts.py` |
| "Next action" panel and wording | `src/hedge_engine/dashboard/next_action.py` |
| Position-control presentation | `src/hedge_engine/dashboard/position_control.py` |
| Theme/styling | `src/hedge_engine/dashboard/theme_colorful.py` |
| Runtime snapshot -> dashboard data -> publication | `src/hedge_engine/runtime/paper_loop.py` |
| Strategy decisions | **not a dashboard change**; strategy/domain engine such as `hybrid_v2.py`, execution and runtime modules |
| Service lifecycle | `deploy/systemd/hedge-engine-paper.service` |

## 4.3 Required tests

Relevant protection includes:

```text
tests/test_dashboard_next_action.py
tests/test_dashboard_position_charts.py
tests/test_dashboard_position_control.py
tests/test_paper_runtime.py
tests/test_paper_runtime_corrections.py
tests/test_operational_safety_v2.py
tests/test_hybrid_v2.py
tests/test_broker_mcx.py
```

A pure theme/layout change should still prove the renderer contract. Any
next-action or position-control change must run the related domain/safety tests
because the displayed instructions are decision-relevant.

## 4.4 Repository recommendation

**Keep D02 in `hedge-engine`.**

It is already correctly bounded: the renderer consumes hedge-engine state and
the paper runtime publishes the report. Moving it to a generic dashboard repo
would either duplicate domain objects or require a new API/data contract with no
current benefit.

What should change instead is the **shared serving layer**, not D02 ownership.

---

# 5. D03 — Live ETF/Kotak Operator Dashboard

## 5.1 Ownership

D03 is not a passive research dashboard. It is an operator surface over live
broker state and guarded live order functions.

```text
Host:
  lakshmidevi

Process:
  src.dashboard.app_pro:app
  Uvicorn 127.0.0.1:8080

Repository:
  kriskingg/etf-invest-engine

Intended production authority:
  main

Observed deployed checkout on 2026-09-19:
  platform-v2-v03-preimplementation-20260916
  306bd5c20dc48c682dab8c85ce1b2c677f97f66a
```

**Change precondition:** reconcile the deployed feature branch with the weekday
08:57 IST hard reset to `origin/main` before modifying production dashboard
code.

## 5.2 If you want to change D03, edit this

| Desired change | Primary source |
|---|---|
| FastAPI composition / which routers and UI injectors are active | `src/dashboard/app_pro.py` |
| Authentication, broker proxy, holdings/orders, live order gates, root page | `src/dashboard/app_hardened.py` |
| Main legacy/root HTML template | `src/dashboard/templates/index.html` |
| Professional UI composition | `src/dashboard/professional_ui.py` |
| Browser/runtime UI patches | `src/dashboard/ui_runtime_fix.py` |
| CNC/MTF sell safety | `src/dashboard/safety.py` |
| Position/broker quote reconciliation | `src/dashboard/position_reconciliation.py` and `position_reconciliation_ui.py` |
| Derivatives | `src/dashboard/derivatives.py`, derivative safety/template modules |
| Basket workflows | `basket.py`, `basket_ui.py`, `basket_resolver.py`, `basket_safety.py` |
| ETF/futures conversion | active UI/backend uses conversion-v2 modules; legacy conversion API retained for compatibility |
| Silver automation/reference UI | `silver_automation.py`, `silver_automation_ui.py` |
| Startup/ingress behavior | `scripts/start_dashboard.sh`, status/stop/restart scripts |

The composition root is the first file to inspect before changing any D03
feature because it determines which implementation is currently wired.

## 5.3 Required tests

The dashboard test family on the deployed feature branch includes:

```text
test_dashboard_hardened_app.py
test_dashboard_safety.py
test_order_safety.py
test_dashboard_derivative_safety.py
test_dashboard_professional_ui.py
test_dashboard_ui_runtime_fix.py
test_position_reconciliation.py
test_position_reconciliation_ui.py
test_basket_api.py
test_basket_resolver.py
test_basket_safety.py
test_basket_ui.py
test_conversion_v2.py
test_dashboard_conversion*.py
test_silver_automation_dashboard.py
```

Run the full ETF dashboard/safety suite for any backend or order-path change.
Never use a live broker order as a deployment test.

## 5.4 Repository recommendation

**Keep D03 in `etf-invest-engine`.**

Reason: its correctness depends directly on live ETF broker/session,
holdings/position reconciliation, MTF/CNC safety, campaign/accounting semantics
and server-side order gates. Extracting the dashboard backend now would create a
cross-repository safety boundary and increase the chance of duplicate rules.

If separation is desired later, first create a versioned internal API that owns
all broker/order safety inside the ETF engine. Only a presentation client should
be separable.

---

# 6. D04 — Platform-v2

## 6.1 Current architecture

D04 is a distinct React/TypeScript/Vite presentation surface but shares the D03
FastAPI process and broker/domain backend.

Frontend source on the deployed feature branch:

```text
web/
  src/App.tsx
  src/pages.tsx
  src/api.ts
  src/types.ts
  src/styles.css
  src/TradeContext.tsx
  src/silverReady.ts
  src/components/
    ObservedBrokerSilver.tsx
    SilverV2Workspace.tsx
    StatusStrip.tsx
    TradeDrawer.tsx
  src/layout/AppShell.tsx
  package.json
  vite.config.ts
```

Built production assets:

```text
web/dist/
```

Do not edit `web/dist` manually.

Backend source:

```text
src/platform_v2/
  api_router.py
  broker*.py
  broker_sync.py
  operational_store.py
  exposure.py
  execution.py
  quality.py
  silver_shadow.py
  silver_ready_router.py
  ...domain/service modules
```

Integration/mount:

```text
src/dashboard/app_pro.py
  -> includes Platform-v2 routers
  -> calls install_spa(app, WEB_DIST)

src/dashboard/spa.py
  -> mounts /v2 from web/dist
  -> provides browser SPA deep-link fallback
```

## 6.2 If you want to change D04, edit this

| Desired change | Primary source |
|---|---|
| Navigation/routes | `web/src/App.tsx` |
| Pages/data presentation | `web/src/pages.tsx` |
| Silver V2 workspace | `web/src/components/SilverV2Workspace.tsx`, related components, `silverReady.ts` |
| Layout | `web/src/layout/AppShell.tsx` |
| API calls from browser | `web/src/api.ts` |
| Type contracts | `web/src/types.ts` |
| CSS | `web/src/styles.css` |
| Backend `/api/v2` behavior | `src/platform_v2/api_router.py` plus the relevant domain/service module |
| Broker truth/normalization | Platform-v2 broker/query/quote modules; do not duplicate browser-side |
| Operational SQLite | `src/platform_v2/operational_store.py` and service users |
| D04 mount/deep links | `src/dashboard/spa.py`; composition in `app_pro.py` |

## 6.3 Required tests

The feature branch contains a broad Platform-v2 contract suite including:

```text
test_platform_v2_api_router.py
test_platform_v2_broker_sync.py
test_platform_v2_broker_truth_ui.py
test_platform_v2_contract_registration.py
test_platform_v2_contracts.py
test_platform_v2_domain.py
test_platform_v2_exposure.py
test_platform_v2_foundation.py
test_platform_v2_instrument_search.py
test_platform_v2_integrity.py
test_platform_v2_ledger_service.py
test_platform_v2_market_data_service.py
test_platform_v2_performance_service.py
test_platform_v2_reservation_service.py
test_platform_v2_risk_workflow_scenario.py
test_platform_v2_scrip_master_status.py
test_platform_v2_silver_ready.py
test_platform_v2_silver_shadow.py
test_platform_v2_sqlite_services.py
test_platform_v2_trade_preview.py
test_platform_v2_v03_architecture.py
```

The D04 browser client must never become the authority for broker truth or
execution safety.

## 6.4 Current documentation/code discrepancy

`web/README.md` still describes the web shell as not connected to existing live
dashboard/broker execution routes, but the deployed branch's
`src/dashboard/app_pro.py` now includes Platform-v2 routers and mounts the SPA
into the live FastAPI process.

Treat code/tests as authority and update stale documentation before using it as
an architecture contract.

## 6.5 Repository recommendation

**Do not split D04 yet.**

First:

1. reconcile `platform-v2-v03-preimplementation-20260916` with the intended
   production branch;
2. establish the final `/api/v2` contract;
3. decide whether D04 replaces, complements or remains a specialist view beside
   D03;
4. prove D03/D04 feature coverage and safety parity;
5. make D04 build/deploy deterministic.

After that, the **React frontend only** is a reasonable candidate for a separate
repo/package if independent release cadence is valuable. The broker/order/safety
backend should remain with the ETF domain unless a deliberately versioned
service boundary is created.

---

# 7. G01 — Shared D01/D02 research serving layer

## 7.1 Current runtime

```text
/opt/chartink-paper/mcx-paper-research/mcx_paper/control_server.py
  -> SimpleHTTPRequestHandler
  -> report directory /var/lib/chartink-paper/nifty-multi-shadow/reports
  -> GET /api/mcx-paper-control
  -> POST /api/mcx-paper-control
```

The current accessible GitHub repositories do not expose a proven authoritative
copy of this runtime file. Therefore this is an **ownership/version-control gap**.

It currently mixes two responsibilities:

1. read-only serving of D01 and D02 generated HTML;
2. mutating MCX paper control.

The public Cloudflare Quick Tunnel exposes the same HTTP origin. The POST checks
a fixed `X-Paper-Control: local-dashboard` value and localhost Origin rules,
not secret-based authentication.

## 7.2 Recommended target

If D01 and D02 continue to share one browser endpoint, create a small
version-controlled **read-only research dashboard gateway**.

Its permitted responsibilities:

- serve generated HTML/assets;
- health endpoint;
- explicit access control / ingress integration;
- no strategy imports;
- no broker credentials;
- no mutation of NIFTY/MCX state.

MCX start/stop/control belongs with `hedge-engine` and should be private and
properly authenticated if retained.

This split removes the odd situation where an old module named
`mcx_paper.control_server` is the public gateway for both independent systems.

---

# 8. Recommended target repository layout

## Decision A — extract NIFTY research

**Recommended, not yet executed.**

Proposed repo:

```text
kriskingg/nifty-options-paper-research
```

Move the coherent NIFTY research application/history there. Keep the existing
OCI path unchanged during the first cutover if possible so repo migration does
not simultaneously become a systemd/path migration.

## Decision B — keep hedge-engine intact

**Recommended.**

D02 and its renderer remain in `kriskingg/hedge-engine`.

## Decision C — keep live ETF backend with ETF engine

**Recommended.**

D03 and D04 broker/safety/backend logic remain in
`kriskingg/etf-invest-engine`.

## Decision D — D04 frontend split is optional/later

**Defer.**

Only consider a separate React frontend repo after the API and production branch
are stable.

## Decision E — replace shared research gateway

**Recommended.**

Either create a dedicated read-only gateway repo/service, or choose one clearly
owned read-only serving implementation. Do not leave mutating MCX control on the
same Internet-facing origin.

## Decision F — Dashboards repo remains documentation only

**Recommended.**

Do not copy D01-D04 runtime source into this repository. Duplication would make
the ownership problem worse.

---

# 9. Safe migration sequence with minimum runtime impact

## Phase 0 — freeze and identify

- no strategy changes;
- no risk/formula changes;
- no live broker changes;
- record current SHAs and runtime hashes;
- fix only correctness/security defects through their authoritative domain repo.

## Phase 1 — NIFTY repository extraction without deployment cutover

1. create the new repo from the authoritative NIFTY subtree with history;
2. reproduce tests from a clean clone;
3. preserve branch/tag/evidence-history references;
4. compare file hashes against the existing authoritative subtree;
5. **do not change chartink-paper services yet**.

Acceptance: source/test parity with no runtime change.

## Phase 2 — NIFTY deployment normalization

1. stage the new checkout separately;
2. verify exact code hashes and tests;
3. preserve existing `/var/lib/chartink-paper` data;
4. choose whether to retain `/opt/chartink-paper/nifty-options-v2` as the
   runtime path initially;
5. update one deployment boundary at a time;
6. verify D01 generated HTML parity and paper-only invariants.

Do not combine this with strategy tuning.

## Phase 3 — shared gateway separation

1. capture/version the current serving contract;
2. build a read-only gateway;
3. serve D01 and D02 files from their existing generated locations;
4. prove byte-identical/static delivery before ingress cutover;
5. move/replace MCX control with a private authenticated domain-owned control
   mechanism;
6. switch ingress only after local parity;
7. retire legacy serving code only after dependency proof.

## Phase 4 — D03/D04 production authority

1. reconcile lakshmidevi deployed branch versus `main`;
2. remove the risk of an unintended weekday hard reset;
3. establish reboot-persistent dashboard startup;
4. compare D03 and D04 feature coverage;
5. decide D04's long-term role;
6. only then consider a frontend-only repo extraction.

---

# 10. Change-control checklist for every dashboard request

Before implementing a dashboard change:

1. identify D01/D02/D03/D04;
2. read this document and that dashboard's detailed page;
3. confirm current Git branch and deployed SHA;
4. identify whether the request is **presentation**, **data contract**,
   **strategy/domain logic**, **serving/ingress**, or **deployment**;
5. edit only the authoritative source, never a generated HTML/dist artifact;
6. run the dashboard-specific tests plus domain/safety tests when applicable;
7. compare generated output;
8. deploy without changing unrelated services;
9. verify local origin first, then private/public ingress;
10. record the new source/deployed SHA and any architecture change back in this
    registry.

A dashboard change is not complete until source ownership, tests, runtime and
registry documentation agree.
