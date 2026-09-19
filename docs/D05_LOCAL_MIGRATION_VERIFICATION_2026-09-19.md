# D05 Local Migration Verification — 2026-09-19

## Scope

This document records the verified local Windows migration of D05 — Mutual Funds Analytics / Tactical Research — from its legacy workspace into the canonical Git working-copy location.

This was a **local filesystem/runtime migration only**. It did not authorize broad Git staging or pushing of generated data.

## Canonical result

```text
Repository:        kriskingg/mf-analytics-source
Branch:            main
Verified HEAD:     5366b9000d0f93af8e4e635b4fc551f0a85d0915
Canonical path:    D:\Git_repos\mf-analytics-source
Frontend:          http://127.0.0.1:5173
Backend docs:      http://127.0.0.1:8000/docs
```

The prior runtime/workspace was:

```text
D:\Dhan\Mutual_funds
```

It is no longer the canonical runtime/code authority.

## Git_repos path correction

Before final migration, `D:\Git_repos` was discovered to be a Windows junction targeting:

```text
D:\AI_Software_Projects\Git_repos
```

That aliasing was intentionally removed so Git repositories would not remain mixed with the AI software workspace.

Final verified identity:

```text
D:\Git_repos
Attributes : Directory
LinkType   :
Target     :
```

Therefore `D:\Git_repos` is now a physical directory, not a junction.

The old `D:\AI_Software_Projects\Git_repos` tree was copied/verified into the new physical location and then renamed as a temporary migration safety backup. No reverse junction is part of the desired final architecture.

## Copy and preservation evidence

During recovery/consolidation:

- a full `robocopy` merge from the old physical Git repository tree into `D:\Git_repos` completed with:
  - 54,558 files;
  - 3.277 GB;
  - 0 failed files;
  - 0 mismatches;
- a subsequent source-to-destination presence check reported:
  - `Missing files from destination: 0`;
- repository-by-repository HEAD checks matched for the valid Git repositories inspected, apart from the old partially moved Alpha remnant;
- the new `D:\Git_repos\Alpha` repository was valid and matched GitHub HEAD `6cf7aa5daca94c9e281e4bd4efc7c5481698c06a`.

## D05 source-integrity evidence

For D05 specifically:

```text
old HEAD: 5366b9000d0f93af8e4e635b4fc551f0a85d0915
new HEAD: 5366b9000d0f93af8e4e635b4fc551f0a85d0915
branch:   main
```

A SHA-256 comparison across Git-tracked D05 files reported:

```text
D05 tracked differences between old and new: 0
```

This verifies preservation of the tracked application source during migration.

## Python environment correction

The inherited virtual environment resolved through the former junction target, so it was intentionally replaced.

A fresh environment was created at:

```text
D:\Git_repos\mf-analytics-source\.venv
```

Verified:

```text
Executable = D:\Git_repos\mf-analytics-source\.venv\Scripts\python.exe
Prefix     = D:\Git_repos\mf-analytics-source\.venv
```

Dependencies were installed from:

```text
app\backend\requirements.txt
```

## Runtime verification

After migration, `app\scripts\start_platform.ps1` started the platform successfully.

Observed checks:

```text
BACKEND:  HTTP 200  -> http://127.0.0.1:8000/docs
FRONTEND: HTTP 200  -> http://127.0.0.1:5173
```

Therefore D05 was operational from the new canonical physical path at the end of the migration verification.

## Data and Git safety

The local D05 tree contains or may contain substantial non-source material, including:

- NAV/history data;
- market/bhavcopy data;
- generated technical datasets;
- browser/test profiles;
- logs;
- `.venv`;
- `node_modules`;
- build output;
- ZIP deliverables/backups;
- reports and validation artifacts.

These may remain local and are not automatically Git content merely because they are physically under the repository directory.

Until the staging/ignore audit is complete:

```text
DO NOT use: git add .
DO NOT use: git add -A
DO NOT bulk-push generated/runtime data
```

Use explicit source-file staging after reviewing `.gitignore` and untracked/non-ignored files.

## Safety backups

Temporary migration safety copies were intentionally retained during verification. They are **not** authoritative runtime locations and should be removed only after final cleanup approval.

Known migration-era backup naming included:

```text
D:\AI_Software_Projects\Git_repos_MIGRATION_BACKUP_20260919
D:\Dhan\Mutual_funds_MIGRATION_BACKUP
D:\Git_repos\mf-analytics-source_CLEAN_GITHUB_BACKUP
```

Their continued existence should be checked before cleanup; this document does not claim they have already been deleted.

## Current authority

For future D05 work, use:

```text
D:\Git_repos\mf-analytics-source
```

and the GitHub repository:

```text
kriskingg/mf-analytics-source
```

Do not use `D:\Dhan\Mutual_funds` or any migration backup as the source of truth.
