# D04 — ETF Dashboard `/v2/` Route

**Status:** ROUTE IDENTITY PENDING RUNTIME PROOF  
**Known URL:** `https://etf-trader.tailabfd53.ts.net/v2/`  
**Server family:** private `etf-trader` / `lakshmidevi` dashboard host is the expected ingress owner, but the exact `/v2/` backend must be traced from the running server.

## Why D04 is separate

A common hostname does not prove that two paths are produced by the same application.

The root dashboard at:

```text
https://etf-trader.tailabfd53.ts.net/
```

is strongly documented as:

```text
Tailscale Serve
 -> http://127.0.0.1:8080
 -> src.dashboard.app_pro:app
 -> kriskingg/etf-invest-engine
```

However, current GitHub source review of `src/dashboard/app_pro.py` does not by itself prove what the deployed `/v2/` URL does. It may be a separate path proxy, legacy alias, redirect, static deployment, or a route supplied by code not obvious from repository keyword search.

Therefore D04 is intentionally not merged into D03 without runtime evidence.

## Required runtime trace

The running `etf-trader` server must answer these questions:

1. What does Tailscale Serve map `/v2/` to?
2. Does `http://127.0.0.1:8080/v2/` return the same content, a redirect, or 404?
3. Is another local port/process responsible?
4. What process owns the route?
5. What is the process cwd and executable?
6. Which file/module/template generates the response?
7. Which Git checkout owns that file?
8. What branch and commit are deployed?
9. Do deployed bytes match that Git commit?
10. Is `/v2/` still a live supported dashboard at all?

## Evidence rule

D04 must not be labelled `PROVEN` merely because the URL loads or shares the `etf-trader.tailabfd53.ts.net` hostname.

The complete proof must be:

```text
URL
 -> Tailscale Serve route
 -> local origin
 -> process
 -> source/generator
 -> checkout
 -> Git remote
 -> branch/commit
 -> deployed-byte comparison
```

## Active audit

A read-only Antigravity audit was dispatched:

```text
DASHBOARD-ETF-RUNTIME-VERIFY-20260919-002
```

It is explicitly instructed to test `/` and `/v2/` independently and record redirects, path-specific Serve mappings, local ports, processes, source paths and byte equivalence.

Until that audit returns, D04 remains:

```text
PARTIAL / ROUTE OWNER UNMAPPED
```
