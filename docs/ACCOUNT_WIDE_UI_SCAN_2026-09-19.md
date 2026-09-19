# Account-Wide Dashboard / Web-UI Scan — 2026-09-19

## Scope

The authenticated GitHub account scan covered **36 accessible repositories** owned by `kriskingg`.

Method:

1. list every accessible repository;
2. inspect each default-branch tree for dashboard/web/UI indicators;
3. inspect obvious application entry points and README/deployment files;
4. perform an account-wide branch-name pass so non-default dashboard branches are not silently missed;
5. classify findings as:
   - **dashboard** — an analytics/operator/portfolio surface that belongs in this registry;
   - **web application / utility** — substantial UI, but not a dashboard;
   - **prototype** — experimental UI, not an operated dashboard;
   - **not a UI system**.

This is a GitHub/source inventory. It does not prove that a local application is currently running unless runtime evidence is separately available.

## Result

### Confirmed dashboards

The account contains **6 confirmed dashboard systems**:

| ID | Dashboard | Repository | Runtime location |
|---|---|---|---|
| D01 | NIFTY Paper Research | `kriskingg/etf-invest-engine`, NIFTY research branch | `chartink-paper` OCI |
| D02 | MCX / Silver Hedge Research | `kriskingg/hedge-engine` | `chartink-paper` OCI |
| D03 | ETF / Kotak Private Operator | `kriskingg/etf-invest-engine` | `lakshmidevi` OCI |
| D04 | ETF Platform-v2 | `kriskingg/etf-invest-engine` | `lakshmidevi` OCI |
| D05 | Mutual Funds Analytics / Tactical Research | `kriskingg/mf-analytics-source` | local Windows, canonical `D:\\Git_repos\\mf-analytics-source` |
| D06 | Personal Investment Tracker | `kriskingg/investment-tracker-app` | local Windows, localhost-only |

### Additional web applications / UI tools not counted as dashboards

#### W01 — KnowYourPNL Web

Repository:

```text
kriskingg/KnowYourPnl-Web
```

Classification:

```text
WEB APPLICATION / PRODUCT
NOT COUNTED AS A DASHBOARD
```

It contains a React/Vite public product with Home, calculator, broker comparison,
ledger, tariff history, methodology, blog and legal pages plus a private
FastAPI calculation API. A zero-cost OCI deployment design exists under
`deploy/`, but no live URL/runtime was proven in this scan.

If the registry is later expanded from "dashboards" to "all web applications",
this should receive its own W01 page.

#### InviCard

Repository:

```text
kriskingg/invicard
```

Classification:

```text
WEB APPLICATION / INVITATION BUILDER
NOT A DASHBOARD
```

Next.js application for invitation templates, canvas editing, publishing and
sharing.

#### python_projects

Repository:

```text
kriskingg/python_projects
```

Classification:

```text
FLASK UTILITY
NOT A DASHBOARD
```

Contains a CSV/text deduplication web utility.

#### Vault_OSS

Repository:

```text
kriskingg/Vault_OSS
```

Classification:

```text
SMALL VAULT ROLE VIEWER UI
NOT A DASHBOARD
```

The template exposes login/token/role-list functions. It is a focused utility,
not an analytics/operator dashboard.

#### my_ideas — MTF-Burn Auditor starter

Repository:

```text
kriskingg/my_ideas
```

Classification:

```text
STATIC PROTOTYPE
NOT AN OPERATED DASHBOARD
```

Contains an HTML/JavaScript starter for an MTF financing/burn calculator.

#### ai-bridge-control

Repository contains dashboard-related job/patch artifacts from engineering work,
but no separate dashboard application was found on its default branch.

## Existing dashboard-feature branches do not create extra dashboards

`etf-invest-engine` contains many historical/feature branches such as:

```text
dashboard-*
silver-automation-dashboard-*
silver-shadow-dashboard-*
tailscale-private-dashboard-*
platform-v2-*
chatgpt/nifty-research-dashboard-v2-plan
```

These map to development history/features of D01, D03 or D04. They are **not
additional current dashboard URLs**.

Likewise, hedge-engine feature branches belong to the D02 domain rather than
representing separate dashboard systems.

## Current count

```text
Confirmed dashboards: 6

OCI dashboards:
  D01
  D02
  D03
  D04

Local Windows dashboards:
  D05
  D06

Substantial web app not counted:
  W01 KnowYourPNL Web
```

## Re-scan rule

Repeat this account-wide scan when:

- a new repository is created;
- a new frontend/server is introduced;
- a new public/private URL is deployed;
- a feature branch creates a genuinely separate operator/research surface;
- a prototype is promoted into an operated application.

Do not increment the dashboard count merely because a new route, tab, feature
branch or component is added to an existing dashboard.


## D05 post-scan migration update — 2026-09-19

After this account-wide source scan, D05 was safely consolidated from the legacy `D:\\Dhan\\Mutual_funds` workspace into the canonical physical Git working copy at `D:\\Git_repos\\mf-analytics-source`. Git branch/HEAD were preserved, tracked old-versus-new D05 content compared with zero differences, the Python environment was rebuilt at the new location, and backend/frontend HTTP 200 checks passed. See [D05_LOCAL_MIGRATION_VERIFICATION_2026-09-19.md](D05_LOCAL_MIGRATION_VERIFICATION_2026-09-19.md).
