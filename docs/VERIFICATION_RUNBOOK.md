# Dashboard Verification Runbook

Use these read-only checks before editing or deploying any dashboard. The objective is to prove the complete path from browser URL to source code.

## 1. Identify the public/private ingress

For Cloudflare quick tunnels:

```bash
ps -ef | grep -i '[c]loudflared'
sudo ss -ltnp | grep cloudflared
```

If the process writes to a rotated log, inspect its open file descriptors rather than restarting it:

```bash
PID=<cloudflared-pid>

sudo ls -l /proc/$PID/fd

sudo sh -c '
for f in /proc/'"$PID"'/fd/*; do
  grep -aEo "https://[a-zA-Z0-9.-]+\.trycloudflare\.com" "$f" 2>/dev/null
done
' | sort -u
```

Do not restart a quick tunnel merely to rediscover its hostname; the hostname can change.

## 2. Identify the listening service

```bash
sudo ss -ltnp
SYSTEMD_PAGER=cat sudo systemctl list-units --type=service --state=running
```

For a known unit:

```bash
SYSTEMD_PAGER=cat sudo systemctl cat <service-name>
sudo systemctl status <service-name> --no-pager -l
```

Record:

- user/group;
- working directory;
- ExecStart;
- environment/PYTHONPATH;
- served directory;
- read/write paths;
- port/bind address.

## 3. Verify what the web server actually serves

Use localhost first:

```bash
curl -I http://127.0.0.1:<port>/<page>
curl -s http://127.0.0.1:<port>/<page> | grep -Eo '<unique-ui-string>' | head
```

This distinguishes deployment problems from browser/ingress problems.

## 4. Inspect the generated artifact

Prefer metadata and targeted string checks. Do not dump large single-line HTML into the terminal.

```bash
sudo stat <generated-file>

sudo -u <service-user> grep -o '<unique-ui-string>' <generated-file> | head
```

Avoid:

```bash
head -80 giant-generated-dashboard.html
```

Some generated dashboards contain megabytes of JSON/JavaScript on a single line.

## 5. Find the generator, not merely the served file

Search source trees with filenames-only output:

```bash
sudo grep -RIl \
  --include='*.py' \
  --include='*.sh' \
  --include='*.js' \
  --exclude-dir=.venv \
  --exclude-dir=reports \
  '<distinctive-ui-string>' \
  /opt /home/<user> 2>/dev/null
```

Useful distinctive strings include:

- page headings;
- unusual button text;
- JavaScript function names;
- generated data-variable names;
- exact output filenames.

## 6. Prove Git ownership

From the runtime checkout:

```bash
git remote -v
git branch --show-current
git rev-parse HEAD
git log -1 --oneline
git status --short
```

Do not assume that a directory under `/opt` is a Git checkout. Confirm the presence of `.git`.

## 7. Prove runtime source equals Git-tracked source

Hash the runtime file:

```bash
git hash-object <runtime-file>
```

Compare that hash with the blob SHA of the intended file in GitHub.

Matching hashes prove byte-for-byte identity at that point in time.

If the runtime file is untracked but corresponds to a tracked source file elsewhere in the repository, document both paths explicitly.

## 8. Dirty-worktree safety check

A dirty deployment checkout is a major safety signal.

Before any deployment action:

```bash
git status --short
git diff --stat
git diff
```

Do not run destructive cleanup commands until every modified/deleted/untracked category has been classified.

Particularly dangerous commands:

```text
git reset --hard
git clean -fd
git checkout .
git switch <branch> without review
blind git pull
```

## 9. Record the result

For every dashboard, record:

```text
URL
ingress/tunnel
server
port
HTTP service
web root
served/generated file
generator service
generator runtime path
Git remote
active branch
active commit
authoritative source path
runtime source path
blob-hash comparison
worktree state
verification date
confidence
```

Use one of these confidence states:

- **PROVEN** — request path and source ownership verified, ideally including blob-hash equality;
- **PARTIAL** — some layers proven, but source/deployment ownership incomplete;
- **UNMAPPED** — URL known but ownership not verified.

## 10. Security rule

This repository is an ownership/runbook registry, not a secret store.

Never commit:

- broker access/session tokens;
- API keys;
- passwords or MPINs;
- TOTP seeds;
- cookies;
- private SSH keys;
- GitHub PATs;
- private credentials;
- secret environment-file contents.

Paths, service names, branch names, public URLs, commit SHAs, and non-secret architecture are acceptable when intentionally documented.
