# D04 — ETF Dashboard `/v2/`

**Status:** ROUTE OWNERSHIP PROVEN; SAME PROCESS AS D03; DISTINCT FRONTEND  
**Runtime verification:** 2026-09-19  
**URL:** `https://etf-trader.tailabfd53.ts.net/v2/`

## Ownership conclusion

D04 is not a second strategy engine and not a separate Tailscale proxy target.

It is a compiled web frontend mounted by the same FastAPI application that serves
D03:

```text
Tailscale Serve
  -> 127.0.0.1:8080
  -> Uvicorn
  -> src.dashboard.app_pro:app
       |
       +-- /      existing Kotak Neo live dashboard
       |
       +-- /v2/   StaticFiles(web/dist, html=True)
```

## Verified source path

```text
/home/ubuntu/kotak/src/dashboard/app_pro.py
/home/ubuntu/kotak/web/dist/index.html
/home/ubuntu/kotak/web/dist/assets/index-CDny3836.js
/home/ubuntu/kotak/web/dist/assets/index-BquxAb8W.css
```

Observed route declaration:

```python
app.mount("/v2", StaticFiles(directory=str(WEB_DIST), html=True), name="platform_v2_web")
```

The same application also includes Platform-v2 API routers.

## Runtime process

D04 uses the same live Uvicorn process as D03:

```text
/home/ubuntu/.venvs/dashboard/bin/uvicorn
src.dashboard.app_pro:app
--host 127.0.0.1
--port 8080
--workers 1
```

## Network boundary

D04 inherits the same private ingress as D03:

```text
https://etf-trader.tailabfd53.ts.net/v2/
  -> Tailscale Serve (tailnet only)
  -> http://127.0.0.1:8080
```

No Cloudflare process/service was observed on lakshmidevi during the audit.

## Relationship to D03

The two surfaces are different presentations inside one application:

- D03 root title: `Kotak Neo | Live Trading & Portfolio Dashboard`;
- D04 title: `Trading Platform`;
- D04 loads compiled assets from `web/dist`.

This proves route/process ownership, but it does not yet prove that D04 is
functionally redundant with D03.

## Cleanup rule

Do not delete or redirect `/v2/` until:

1. feature coverage is compared against D03;
2. any Platform-v2 APIs/state used only by D04 are identified;
3. operator usage is established;
4. rollback is defined.

See [LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md](LAKSHMIDEVI_RUNTIME_VERIFICATION_2026-09-19.md).
