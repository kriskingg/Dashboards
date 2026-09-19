# Dashboard Repository Boundary Decision Register

**Status:** decisions proposed from verified 2026-09-19 runtime/source mapping.  
**No migrations have been executed by this document.**

| Decision | Current state | Proposed state | Why | Blockers before execution | Status |
|---|---|---|---|---|---|
| NIFTY paper research ownership | subtree/non-main branch inside `etf-invest-engine` | dedicated `nifty-options-paper-research` repo | independent paper-research domain, tests, deploy and data lifecycle | preserve history/evidence branch; normalize hybrid OCI layout; no strategy behavior drift | **RECOMMENDED** |
| MCX/Silver dashboard | `hedge-engine` | remain in `hedge-engine` | renderer and runtime consume same hedge domain state | none for ownership; fix serving boundary separately | **KEEP** |
| Live ETF D03 backend/UI | `etf-invest-engine` | remain in `etf-invest-engine` | live broker/order safety is domain-coupled | reconcile deployed branch vs main; reboot startup | **KEEP** |
| Platform-v2 backend | `etf-invest-engine` feature branch | remain with ETF domain initially | broker truth, operational store and execution/safety contracts | production branch/API stabilization | **KEEP FOR NOW** |
| Platform-v2 React frontend | `web/` in ETF repo feature branch | optional separate frontend repo/package later | static client could have independent release cadence | stable versioned API, parity tests, deterministic build/deploy | **DEFER** |
| Shared research web server | runtime `mcx_paper.control_server`, Git authority unproven | versioned read-only gateway | D01/D02 share ingress but not domain ownership | capture authoritative source; remove mutating control from public origin | **RECOMMENDED** |
| MCX paper control | public shared origin | private authenticated hedge-domain control | current fixed-header/Origin check is not strong auth | define control contract and private access | **SECURITY BLOCKER** |
| Mutual Funds Analytics D05 | `mf-analytics-source` | remain in `mf-analytics-source` | coherent analytics/data/UI domain already separated | keep local runtime docs current | **KEEP** |
| Investment Tracker D06 | `investment-tracker-app` | remain in `investment-tracker-app` | clean independent local-first accounting app | keep port/runtime docs current | **KEEP** |
| `Dashboards` repo | ownership docs | remain docs/decision registry | one control plane without copying runtime source | keep links/current SHAs refreshed | **KEEP** |

## Important distinction: "mixed repo" versus "correct coupling"

Not all code co-location is bad.

- **Bad mixing:** NIFTY paper research and live ETF production sharing a repo
  only through branch history/name while having different servers, execution
  authority, state and deployment contracts.
- **Correct coupling:** D03 broker/order/safety dashboard code living with the
  ETF production domain it controls.
- **Correct coupling:** D02 renderer living with the hedge state machine whose
  state it interprets.
- **Potential future separation:** D04 React client after the backend becomes a
  stable API.

The goal is not one repository per URL. The goal is one clear authority per
domain and no duplicated safety/business logic.
