# Infrastructure Audit Checklist

**Last updated:** 2026-09-19  
**Scope:** `lakshmidevi` and `chartink-paper`  
**Mode:** evidence first; no cleanup or architecture changes during audit.

This is the permanent progress tracker for the two OCI hosts. It distinguishes
facts already proven from questions that still require evidence.

## Host naming

| Host | Public IP | Status |
|---|---|---|
| `lakshmidevi` | `141.148.219.153` | mapped |
| `chartink-paper` | `80.225.234.65` | mapping/security/durability audit in progress |

Do not use ordinal labels such as "Server 1" or "Server 2" in current
documentation. Historical filenames may retain those terms where renaming would
destroy provenance; current prose should identify the host explicitly.

## Lakshmidevi

### Done

- [x] Host/network/listener inventory.
- [x] Tailscale node and Serve mapping.
- [x] Proved Uvicorn is loopback-only on `127.0.0.1:8080`.
- [x] Proved no cloudflared process/service on lakshmidevi.
- [x] Classified D03 `/`.
- [x] Classified D04 `/v2/` as a distinct compiled frontend inside the same
      FastAPI/Uvicorn process.
- [x] Recorded exact deployed ETF checkout branch/HEAD.
- [x] Recorded hedge-engine checkout/service ownership.
- [x] Identified local mutable ETF/dashboard/hedge state.
- [x] Identified no evidenced scheduled off-VM backup for local application data.
- [x] Recorded dashboard reboot-persistence gap.
- [x] Updated `etf-invest-engine` and `Dashboards` documentation.

### Must check before action

- [ ] Reconcile intended live ETF authority (`main`) with observed
      `platform-v2-v03-preimplementation-20260916` checkout at
      `306bd5c20dc48c682dab8c85ce1b2c677f97f66a`.
- [ ] Review the weekday 08:57 IST `git fetch` +
      `git reset --hard origin/main` path before allowing it to redefine the
      deployed checkout.
- [ ] Establish and test deliberate reboot-persistent startup for D03/D04.
- [ ] Define off-VM backup/restore policy for Platform-v2 operational SQLite,
      research SQLite, dashboard config/logs and `/var/lib/hedge-engine`.
- [ ] Compare D03 and D04 feature coverage before considering route retirement.
- [ ] Treat hedge shadow broker-auth failure as an operational-health item only;
      do not alter strategy design as part of infrastructure cleanup.

## Chartink-paper

### Proven before the deep durability pass

- [x] D01 NIFTY generator ownership traced to
      `kriskingg/etf-invest-engine`,
      branch `chatgpt/statistical-options-buying-basket-v1`.
- [x] D02 MCX generator ownership traced to `kriskingg/hedge-engine`.
- [x] Shared HTTP/control server identified as
      `mcx_paper.control_server` on `127.0.0.1:8765`.
- [x] Cloudflare Quick Tunnel identified as the public bridge to port 8765.
- [x] NIFTY executable runtime source proven current for the changed runtime
      source while checkout metadata remains hybrid/stale.
- [x] MCX hedge-engine tracked runtime proven current at the prior verification.
- [x] Historical `mcx-paper-live.service` identified as disabled/inactive.

### Deep audit evidence already captured

- [x] Systemd unit hashes and reconstruction material.
- [x] Actual local archive/backup files inventoried.
- [x] No scheduled application-data backup/export job found in user/root cron.
- [x] NIFTY evidence-exporter timer/service identified as the Git evidence
      publication path.
- [x] OCI CLI is not installed, so OCI boot-volume/Object Storage backup state
      was not proven from the host.
- [x] No world-writable or group-writable entries found under
      `/var/lib/chartink-paper` and `/var/lib/hedge-engine`.
- [x] Journald bounded to approximately 100 MB / 7 days.
- [x] Local deployment/source rollback archives exist, but same-disk archives
      are not counted as VM-loss protection.
- [x] Current failed units observed:
      `nifty-multi-shadow-evidence-exporter.service`,
      `nifty-multi-shadow.service`,
      `nifty-options-v2-replay.service`.

### Final evidence still required from chartink-paper

The uploaded deep-audit transcript begins after the early audit sections, so the
following evidence must be recollected in a small targeted read-only pass:

- [ ] Exact `cloudflared-tunnel.service` unit/startup/restart/enable state.
- [ ] Quick Tunnel hostname history across retained boots/logs; determine whether
      reboot/restart creates a new hostname.
- [ ] Exact `mcx_paper.control_server` HTTP methods/routes.
- [ ] Exact authentication/authorization checks on control endpoints.
- [ ] Determine whether mutating control endpoints are reachable through the
      public Quick Tunnel and what rejects unauthorized requests.
- [ ] Capture safe local/public GET response evidence and security headers.
- [ ] Capture exact status/result/journal failure reason for the three currently
      failed NIFTY units; classify each as active bug, expected one-shot outcome,
      or legacy residue.
- [ ] Capture complete mutable-data root sizes, current important-file inventory,
      recent-write ordering and SQLite schemas in strict read-only mode.
- [ ] Map live process -> currently open mutable files.
- [ ] Confirm mounted filesystem for all mutable roots.
- [ ] Determine whether OCI control-plane backups exist using an external OCI
      control-plane check if host-local proof remains unavailable.
- [ ] Determine whether documented Object Storage/laptop backup cadence is
      actually implemented; documentation alone is not proof.
- [ ] Determine restore-tested status for every dataset.

## Final deliverables after evidence closure

- [ ] Final two-host durability matrix:
      data -> writer -> location -> reboot-safe -> VM-loss-safe -> backup ->
      restore procedure -> restore tested.
- [ ] Public/private exposure matrix.
- [ ] Reboot/startup matrix.
- [ ] Source/deployed-SHA ownership matrix.
- [ ] Data-retention and writer matrix.
- [ ] Documentation-versus-runtime discrepancy list.
- [ ] Cleanup candidate list with dependency proof and rollback requirement.

## Cleanup gate

No deletion, move, rename, service restart, service enable/disable, cron change,
Git reset/pull, Tailscale/Cloudflare change, port change, dashboard retirement,
strategy change, or trading action is authorized until the final evidence and
dependency matrix are complete.
