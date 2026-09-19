# D05 Session Turnover / Continuation Guide — 2026-09-19

> **CRITICAL REPOSITORY GUARD — READ BEFORE DOING ANYTHING**
>
> D05 is **NOT** in `kriskingg/etf-invest-engine`.
>
> For this turnover task, ignore any project-level default that points to the NIFTY/options repository. The authoritative D05 application repository is:
>
> ```text
> kriskingg/mf-analytics-source
> ```
>
> The central documentation/turnover repository is:
>
> ```text
> kriskingg/Dashboards
> ```
>
> If a session starts by inspecting `kriskingg/etf-invest-engine`, stop that path immediately and switch to the two repositories above before making any claim or change.

> **SCOPE NOTE**
>
> This page is D05-specific. For work spanning D01-D06, first read:
> [ALL_DASHBOARDS_SESSION_TURNOVER_2026-09-19.md](ALL_DASHBOARDS_SESSION_TURNOVER_2026-09-19.md).
>
> There are currently **6 confirmed dashboards** in the central registry. Do not treat this D05 page as the handoff for D01-D04 or D06.

## Purpose

This page is the handoff for the next ChatGPT session working on **D05 — Mutual Funds Analytics / Tactical Research**.

The immediate objective is to finish the D05 repository consolidation safely:

1. preserve all real local application source;
2. keep large/local runtime data out of GitHub;
3. migrate all active path references from the legacy workspace to the canonical workspace;
4. combine the preserved local-source branch with the path-cleanup branch;
5. run backend/frontend/runtime validation;
6. merge only the validated combined result to `main`;
7. update the central documentation after final merge;
8. keep migration safety backups until final acceptance.

Do **not** infer current state from old local folders. GitHub code + tests are the source authority for repository content, while the verified local runtime path below is the current workstation authority.

---

## Current system identity

| Item | Current value |
|---|---|
| Dashboard ID | D05 |
| Application | Mutual Funds Analytics / Tactical Research |
| GitHub repository | `kriskingg/mf-analytics-source` |
| GitHub URL | https://github.com/kriskingg/mf-analytics-source |
| Current `main` commit | `5366b9000d0f93af8e4e635b4fc551f0a85d0915` |
| Canonical local workspace | `D:\Git_repos\mf-analytics-source` |
| Local URI | `file:///D:/Git_repos/mf-analytics-source/` |
| Legacy workspace | `D:\Dhan\Mutual_funds` — no longer runtime/code authority |
| Frontend | React + TypeScript + Vite |
| Frontend URL | `http://127.0.0.1:5173` |
| Backend | FastAPI |
| Backend docs URL | `http://127.0.0.1:8000/docs` |
| Database | PostgreSQL |
| OCI dependency | none proven for D05 |

The D05 application was successfully started from the new canonical path and both frontend and backend returned HTTP 200 during migration verification.

---

## Why this work started

The local development estate had Git repositories and application work spread across multiple roots, especially:

- `D:\Dhan`
- `D:\AI_Software_Projects`
- `D:\Git_repos`

The goal is to make `D:\Git_repos` the physical canonical Git workspace while keeping heavy datasets and runtime state local.

During the audit, `D:\Git_repos` was discovered to have been a junction into `D:\AI_Software_Projects\Git_repos`. That aliasing was removed and `D:\Git_repos` is now a real Windows directory.

D05 was then verified under:

```text
D:\Git_repos\mf-analytics-source
```

The old `D:\Dhan\Mutual_funds` path must not become authoritative again.

---

## Verified migration state

The migration verification established:

- the new `D:\Git_repos` root is a physical directory, not a junction;
- old and new D05 tracked source matched during the migration comparison;
- the D05 remote remained `kriskingg/mf-analytics-source`;
- the migrated local repository remained based on `main` commit `5366b9000d0f93af8e4e635b4fc551f0a85d0915`;
- the Python virtual environment was rebuilt under the new canonical path;
- backend `http://127.0.0.1:8000/docs` returned HTTP 200;
- frontend `http://127.0.0.1:5173` returned HTTP 200;
- no running D05 process was found using `D:\Dhan\Mutual_funds`;
- no Scheduled Task action was found using the old D05 path;
- no current environment variable was found using the old D05 path.

Do not recreate a junction from the new workspace back to the legacy path.

---

## Git state that must be preserved

### 1. Main branch

Current GitHub `main`:

```text
5366b9000d0f93af8e4e635b4fc551f0a85d0915
```

Commit:
https://github.com/kriskingg/mf-analytics-source/commit/5366b9000d0f93af8e4e635b4fc551f0a85d0915

### 2. Preserved local-source branch

The meaningful unpublished local source was audited and explicitly preserved.

Branch:

```text
preserve/d05-local-source-20260919
```

Branch URL:
https://github.com/kriskingg/mf-analytics-source/tree/preserve/d05-local-source-20260919

Preservation commit:

```text
478e7a018ba0d587915445d4f7decfb056c63a0d
```

Commit URL:
https://github.com/kriskingg/mf-analytics-source/commit/478e7a018ba0d587915445d4f7decfb056c63a0d

This branch is exactly one commit ahead of `main` and contains the pre-integration local source that must not be lost.

Files preserved in that commit:

```text
README.md
app/assets/app_icon.ico
app/backend/app/mf_signal_crossovers/rsi_ema.py
app/backend/app/routes/funds.py
app/backend/app/scheduler.py
app/backend/scripts/auto_update.py
app/backend/tests/test_audit_regressions_20260911.py
app/backend/tests/test_turnaround_hunter.py
app/frontend/src/App.tsx
app/frontend/src/assets/hero.png
app/frontend/src/chartUxHelpers.ts
app/frontend/src/components/MutualFundChart.tsx
app/frontend/src/services/chartExtrasApi.ts
```

Stats from the GitHub comparison:

- 13 files;
- 398 insertions;
- 119 deletions;
- 10 modified tracked source files;
- 1 new backend regression test;
- 2 new application assets.

Do **not** delete, rewrite or bypass this preservation branch until the combined integration is merged and verified.

### 3. Path-cleanup branch and PR

Branch:

```text
chatgpt/d05-path-cleanup-20260919
```

Branch URL:
https://github.com/kriskingg/mf-analytics-source/tree/chatgpt/d05-path-cleanup-20260919

Current branch head:

```text
a7bd4c0903867f6b5e4d378a502f27480c3813fe
```

Open PR:

https://github.com/kriskingg/mf-analytics-source/pull/1

PR title:

```text
chore: migrate D05 path references to canonical workspace
```

Current PR state at handoff:

- open;
- mergeable;
- not merged;
- base: `main`;
- 67 changed files;
- 73 commits because the GitHub contents API created per-file commits;
- 176 additions;
- 179 deletions.

Key intended changes in PR #1:

- `start_app.bat` delegates to portable `app/scripts/start_platform.ps1`;
- removes old hardcoded `D:\Dhan\.venv` / `D:\Dhan\Mutual_funds` launch assumptions;
- aligns current backend startup/documentation to port 8000;
- `scripts/check_nav_sync_status.py` derives `PROJECT_ROOT` from the script location and uses `PROJECT_ROOT / "data"`;
- `scripts/sync_new_nav_to_postgres.py` derives its data path from the repository root;
- root `run_market_radar_validation_chatgpt.ps1` defaults project root to its own repository path;
- old D05 workspace references were updated across current docs, runnable tooling, prompt packages and tracked historical/disabled files;
- target after integration is **zero tracked references** to `D:\Dhan\Mutual_funds`.

Important: PR #1 must **not** be merged directly before reconciliation with the preserved local-source branch.

---

## Local working-tree state at handoff

The last reported local branch is:

```text
preserve/d05-local-source-20260919
```

with HEAD:

```text
478e7a018ba0d587915445d4f7decfb056c63a0d
```

After preservation:

- `git diff --name-only` was empty;
- `git diff --stat` was empty;
- Git index/stat metadata was refreshed;
- GitHub Desktop's apparent hundreds of modifications collapsed once the index was refreshed;
- stray accidental untracked console-output files were removed;
- the user reported the final unwanted file deleted.

The next session must still begin with a fresh read-only verification:

```powershell
Set-Location "D:\Git_repos\mf-analytics-source"

git branch --show-current
git rev-parse HEAD
git status --short
git --no-pager diff --name-only
git remote -v
```

Expected tracked state before integration: no real tracked working-tree diff.

If this expectation is not true, stop and inspect before merging anything.

---

## Source vs local-data boundary

The agreed target is:

### GitHub should contain

- application source;
- backend and frontend tests;
- scripts;
- SQL/schema files;
- current documentation;
- small required assets;
- small config/templates/reference material where appropriate.

### Local only

- NAV/history datasets;
- bhavcopy/market datasets;
- Parquet feature/history files;
- live PostgreSQL database state;
- logs;
- caches;
- `.venv`;
- `node_modules`;
- build/cache output;
- ZIP deliverables;
- migration backups;
- generated result directories;
- ChatGPT/AG temporary kits/backups.

The D05 audit found **no Git-tracked file larger than 20 MB**.

Representative large local files included Parquet datasets and ZIP deliverables in the 20–50+ MB range. They are intentionally not canonical Git source.

Current repository ignore policy covers the main heavy/runtime classes, including:

```text
.env
.env.*
*.local
venv/
.venv/
node_modules/
dist/
.vite/
data/
*.parquet
*.csv.gz
*.db
*.sqlite
*.sqlite3
*.log
logs/
temp/
tmp/
*.zip
screenshots/
```

Important exception:

```text
*.csv
```

is **not** globally ignored.

Therefore any ordinary CSV must be classified before staging. Some small CSVs are legitimate project/reference artifacts.

Local-only exclusions were also added in `.git/info/exclude` for:

```text
.chatgpt_*_backup/
.chatgpt_*_kit/
results/
```

These are workstation-only exclusions and are intentionally not repository policy.

### Git staging safety

For D05, do not use blind broad staging while local data/generated material is present:

```text
DO NOT default to: git add .
DO NOT default to: git add -A
DO NOT default to: git commit -a
```

Prefer explicit reviewed paths.

---

## Safety backups

The principal migration safety backup that was deliberately retained is:

```text
D:\AI_Software_Projects\Git_repos_MIGRATION_BACKUP_20260919
```

Do **not** delete it yet.

Other migration-era backup names documented during the audit include:

```text
D:\Dhan\Mutual_funds_MIGRATION_BACKUP
D:\Git_repos\mf-analytics-source_CLEAN_GITHUB_BACKUP
```

Their current existence must be re-checked before any cleanup; do not assume they are still present.

Backups are safety copies only. None is the current code/runtime authority.

---

## Central documentation

The central dashboard/ownership registry is:

https://github.com/kriskingg/Dashboards

D05 overview:

https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D05-MUTUAL-FUNDS-ANALYTICS.md

D05 migration evidence:

https://github.com/kriskingg/Dashboards/blob/main/docs/D05_LOCAL_MIGRATION_VERIFICATION_2026-09-19.md

Registry README:

https://github.com/kriskingg/Dashboards/blob/main/README.md

The central D05 documentation was updated through source preservation and merged in Dashboards commit:

```text
7c9366f990562dbc51065a24f4fe93acff2432bc
```

Commit URL:
https://github.com/kriskingg/Dashboards/commit/7c9366f990562dbc51065a24f4fe93acff2432bc

That documentation intentionally omits temporary terminal/index troubleshooting and records only durable state.

---

## What has already been completed

1. Identified D05 as `kriskingg/mf-analytics-source`.
2. Established `D:\Git_repos\mf-analytics-source` as the canonical physical workspace.
3. Removed the `D:\Git_repos` junction architecture.
4. Verified the repository copy during migration.
5. Rebuilt the D05 Python environment under the new path.
6. Started D05 from the new path.
7. Verified frontend HTTP 200 on 5173.
8. Verified backend docs HTTP 200 on 8000.
9. Verified no active process/task/environment reference requires the old runtime path.
10. Audited local tracked and untracked content.
11. Identified the meaningful unpublished local source.
12. Preserved that source in GitHub commit `478e7a0...`.
13. Audited heavy/local data and confirmed it should remain local.
14. Confirmed no tracked file above 20 MB.
15. Established the GitHub-vs-local data boundary.
16. Created path-cleanup PR #1.
17. Cleaned the local tracked Git state after preservation.
18. Updated and merged the central D05 documentation in `kriskingg/Dashboards`.

---


## 2026-09-19 final integration completion

D05 repository consolidation is complete on GitHub.

Final merged authority:

```text
repository: kriskingg/mf-analytics-source
branch: main
main SHA: d25165a31e8a64e8f9814f0f58aa8366ffce2a0c
combined PR: #2
validated integration head: 48ebe6f1f9e2bdfffbbb9356e58ff9e5a362abac
```

GitHub ancestry was verified after merge:
- `main` contains the preserved local-source branch history;
- `main` contains the path-cleanup branch history;
- `main` contains the validated integration head;
- PR #1 is superseded by the combined merge and is no longer pending independently;
- no source/preservation/integration branch was deleted.

Final local validation performed before merge:
- `git diff --check main...HEAD`: PASS;
- tracked `D:\Dhan\Mutual_funds` references: zero;
- backend non-runtime suite: **197 passed, 24 intentionally skipped**;
- lifecycle/runtime suite: **3 passed**;
- frontend production build: PASS;
- frontend lint: **0 errors, 19 warnings**;
- backend `http://127.0.0.1:8000/docs`: HTTP 200;
- frontend `http://127.0.0.1:5173`: HTTP 200;
- running-process legacy path check: zero;
- working tree after validation: clean.

Important test-safety corrections included in the merge:
- updater regression tests no longer reach normal AMFI/database mutation paths;
- lifecycle paths derive from the repository root;
- dynamic clean-equity counts are no longer pinned to stale value 817;
- `app/backend/requirements-test.txt` declares backend test dependencies.

Remaining administrative follow-up:
- switch/pull the local checkout to merged `main` when convenient;
- keep migration backups until explicit cleanup approval;
- do not delete preservation/integration branches solely because the merge completed.

The pre-merge integration procedure below is retained only as historical evidence and is no longer an active instruction.

---

## Historical pre-merge instructions (superseded)

### Highest-priority next task

Create a combined integration branch containing both:

```text
preserve/d05-local-source-20260919
+
chatgpt/d05-path-cleanup-20260919
```

Do not merge either source independently to `main` until the combined result is reviewed and tested.

### Recommended safe integration sequence

Start only after confirming the local working tree is clean.

```powershell
Set-Location "D:\Git_repos\mf-analytics-source"

git fetch origin --prune

git switch preserve/d05-local-source-20260919
git pull --ff-only

git switch -c chatgpt/d05-integration-20260919

git merge --no-ff origin/chatgpt/d05-path-cleanup-20260919
```

If there is a conflict, stop and resolve it deliberately.

Expected likely overlap: `README.md`, because both histories modify it.

Do not resolve a conflict by blindly choosing all of `ours` or all of `theirs`. Preserve the local-source changes **and** the canonical-path/port corrections.

### After merge resolution

Run:

```powershell
git status --short
git --no-pager diff --check
git --no-pager diff --stat
git grep -n -I -F 'D:\Dhan\Mutual_funds' --
```

Acceptance target for the old path:

```text
0 matches
```

Then inspect any remaining absolute `D:\Git_repos\mf-analytics-source` usages. Prefer dynamic path derivation for operational scripts where practical; do not mechanically convert historical prose or packaging evidence without understanding its purpose.

### Backend validation

At minimum, run the affected/new regression tests and then the broader backend suite if practical.

Relevant preservation tests include:

```text
app/backend/tests/test_audit_regressions_20260911.py
app/backend/tests/test_turnaround_hunter.py
```

Use the canonical virtual environment:

```text
D:\Git_repos\mf-analytics-source\.venv\Scripts\python.exe
```

Do not treat “tests passed” as sufficient if code review shows path or source-contract defects.

### Frontend validation

From:

```text
D:\Git_repos\mf-analytics-source\app\frontend
```

run the normal production build and any existing lint/test command defined by `package.json`.

At minimum the Vite production build must succeed before merge.

### Runtime smoke validation

After code/tests/build pass:

- start with `app\scripts\start_platform.ps1`;
- verify backend `http://127.0.0.1:8000/docs` -> HTTP 200;
- verify frontend `http://127.0.0.1:5173` -> HTTP 200;
- confirm the running processes use `D:\Git_repos\mf-analytics-source`;
- confirm no operational path falls back to `D:\Dhan\Mutual_funds`.

### Final GitHub sequence

After successful integration validation:

1. push `chatgpt/d05-integration-20260919`;
2. create a combined PR to `main`;
3. review actual GitHub diff;
4. verify tests/build/runtime evidence;
5. merge the combined PR;
6. close/supersede PR #1 if it is no longer needed independently;
7. keep the preservation branch until post-merge verification is complete;
8. update the central Dashboards D05 docs with the final merged SHA and validation result.

Do not merge PR #1 first merely because GitHub reports it mergeable.

---

## Cleanup that must wait

Do not delete migration backups yet.

Only consider backup retirement after all of these are true:

- combined D05 PR merged;
- local `main` updated to the merged GitHub SHA;
- tracked working tree clean;
- backend tests pass;
- frontend build passes;
- runtime smoke checks pass from the canonical path;
- old-path Git grep returns zero;
- local NAV/bhav/Parquet/PostgreSQL state confirmed intact;
- central documentation updated;
- explicit cleanup approval given.

No destructive `git reset --hard`, `git clean`, force push or broad file deletion should be used as a shortcut.

---

## D06 warning

There is a separate local application:

```text
D:\Git_repos\investment-tracker-app
```

D06 has had local unpublished work in the wider consolidation effort.

Do **not** apply D05 cleanup/reset assumptions to D06. Treat it as a separate preservation/audit task.

D06 documentation:
https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D06-INVESTMENT-TRACKER.md

---

## Operational principles established in this conversation

- Inspect first; preserve before modifying.
- GitHub should contain canonical source, not bulk runtime data.
- Local datasets are not “missing from Git” merely because they are intentionally local.
- Never destroy a local working tree simply to make Git look clean.
- A branch being mergeable is not equivalent to being validated.
- Code + tests outrank stale documentation.
- Documentation should describe durable architecture/state, not temporary troubleshooting.
- Preserve safety backups until the new canonical state has been independently validated.
- For path migrations, prefer dynamic path derivation over new hardcoded absolute paths where the script can be portable.

---

## First message for the next ChatGPT session

The user can simply say:

> Continue D05 from the turnover page. Verify current GitHub state first, then continue with the preservation + path-cleanup integration. Do not merge or delete anything until the combined branch is reviewed and tested.

The next session should read this page first, then inspect the live GitHub refs before giving commands.

---

## Quick-link index

- D05 repository: https://github.com/kriskingg/mf-analytics-source
- D05 main baseline commit: https://github.com/kriskingg/mf-analytics-source/commit/5366b9000d0f93af8e4e635b4fc551f0a85d0915
- Preserved-source branch: https://github.com/kriskingg/mf-analytics-source/tree/preserve/d05-local-source-20260919
- Preserved-source commit: https://github.com/kriskingg/mf-analytics-source/commit/478e7a018ba0d587915445d4f7decfb056c63a0d
- Path-cleanup branch: https://github.com/kriskingg/mf-analytics-source/tree/chatgpt/d05-path-cleanup-20260919
- Path-cleanup PR #1: https://github.com/kriskingg/mf-analytics-source/pull/1
- Central Dashboards registry: https://github.com/kriskingg/Dashboards
- D05 dashboard page: https://github.com/kriskingg/Dashboards/blob/main/docs/dashboards/D05-MUTUAL-FUNDS-ANALYTICS.md
- D05 migration verification: https://github.com/kriskingg/Dashboards/blob/main/docs/D05_LOCAL_MIGRATION_VERIFICATION_2026-09-19.md
- Central documentation state commit: https://github.com/kriskingg/Dashboards/commit/7c9366f990562dbc51065a24f4fe93acff2432bc

Local paths:

```text
Canonical D05:
D:\Git_repos\mf-analytics-source

Canonical D05 local URI:
file:///D:/Git_repos/mf-analytics-source/

Primary retained Git-root migration backup:
D:\AI_Software_Projects\Git_repos_MIGRATION_BACKUP_20260919

Legacy D05 path — no longer authority:
D:\Dhan\Mutual_funds
```

---

## Handoff status

**Current phase:** source preserved; path cleanup prepared; combined integration not yet performed.

**Do next:** verify local/GitHub state, create combined integration branch, resolve overlap, test, smoke-test, review, then merge.

**Do not do next:** merge PR #1 directly, delete migration backups, bulk-stage generated data, or reset/clean the working tree destructively.
