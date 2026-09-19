# Dashboard Pages

This directory contains one canonical decision page per confirmed dashboard. The current account-wide count is **6**.

Use these pages when the question is:

- Which dashboard is this?
- Which server runs it?
- Which repository/branch owns it?
- Which source files should be changed?
- Which tests must be run?
- Which runtime/service will be affected?
- Which data/state does it depend on?
- What are the deployment or migration risks?
- Should the code stay in its current repository?

These pages are intentionally concise operational summaries. Deep audit evidence remains in the existing detailed documents under `docs/`.

| ID | Dashboard | Canonical page | Deep evidence |
|---|---|---|---|
| D01 | NIFTY Paper Research | [D01-NIFTY-PAPER-RESEARCH.md](D01-NIFTY-PAPER-RESEARCH.md) | [../D01-nifty-paper-research.md](../D01-nifty-paper-research.md) |
| D02 | MCX / Silver Hedge Research | [D02-MCX-SILVER-HEDGE.md](D02-MCX-SILVER-HEDGE.md) | [../D02-mcx-silver-hedge.md](../D02-mcx-silver-hedge.md) |
| D03 | ETF / Kotak Operator Dashboard | [D03-ETF-KOTAK-OPERATOR.md](D03-ETF-KOTAK-OPERATOR.md) | [../D03-etf-kotak-private-dashboard.md](../D03-etf-kotak-private-dashboard.md) |
| D04 | ETF Platform-v2 | [D04-ETF-PLATFORM-V2.md](D04-ETF-PLATFORM-V2.md) | [../D04-etf-dashboard-v2.md](../D04-etf-dashboard-v2.md) |
| D05 | Mutual Funds Analytics / Tactical Research | [D05-MUTUAL-FUNDS-ANALYTICS.md](D05-MUTUAL-FUNDS-ANALYTICS.md) | [../ACCOUNT_WIDE_UI_SCAN_2026-09-19.md](../ACCOUNT_WIDE_UI_SCAN_2026-09-19.md) |
| D06 | Personal Investment Tracker | [D06-INVESTMENT-TRACKER.md](D06-INVESTMENT-TRACKER.md) | [../ACCOUNT_WIDE_UI_SCAN_2026-09-19.md](../ACCOUNT_WIDE_UI_SCAN_2026-09-19.md) |

For cross-dashboard decisions, also use:

- [../DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md](../DASHBOARD_CODE_CHANGE_AND_OWNERSHIP_GUIDE.md)
- [../DASHBOARD_REPOSITORY_BOUNDARY_DECISIONS.md](../DASHBOARD_REPOSITORY_BOUNDARY_DECISIONS.md)
- [../MASTER_SYSTEM_MAP.md](../MASTER_SYSTEM_MAP.md)
