# Security and Repository Hygiene

This repository is currently public. Treat every committed file as publicly readable.

## Allowed documentation

It is acceptable to document:

- public dashboard URLs;
- server/service names;
- non-secret filesystem paths;
- Git repository names;
- branch names;
- commit SHAs;
- source-file paths;
- systemd unit names;
- high-level runtime/deployment architecture;
- non-secret verification commands.

## Never commit

Do not store:

- broker access tokens or session tokens;
- API keys;
- passwords, PINs, MPINs;
- TOTP seeds;
- cookies or browser sessions;
- private SSH keys;
- GitHub PATs;
- cloud credentials;
- secret environment-file contents;
- personally identifying account numbers;
- private broker/order credentials.

When server commands display secrets, redact them before adding evidence here.

## Operational safety

This repository is documentation only. It is not a deployment source.

Updating this repository must not:

- restart trading/research services;
- change runtime flags;
- place broker orders;
- alter source repositories;
- modify systemd units;
- clean deployment worktrees;
- change Cloudflare or Tailscale configuration.

Dashboard source changes belong to the repository/branch identified in the corresponding ownership record and should follow that system's own review/deployment process.
