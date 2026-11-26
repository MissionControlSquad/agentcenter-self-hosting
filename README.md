# AgentCenter Self-Hosting

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Validate](https://github.com/MissionControlSquad/agentcenter-self-hosting/actions/workflows/validate.yml/badge.svg)](https://github.com/MissionControlSquad/agentcenter-self-hosting/actions/workflows/validate.yml)

Docker Compose files and a setup guide for running AgentCenter, an AI agent management dashboard,
on your own server.

## Project status

The hosted AgentCenter app service (cloud tenants and the license server) has been shut down, and this repository is now
published under the [MIT License](LICENSE). It is provided as-is and maintained on a best-effort
basis by the community.

No license key is needed. The license check has been removed from the app
([MissionControlSquad/monorepo#165](https://github.com/MissionControlSquad/monorepo/pull/165)), so an
install is fully unlocked from first start and never contacts a license server.

One thing to know before you start: app images (`ghcr.io/missioncontrolsquad/agentcenter`) published
before that change still ask for a license key and can't be activated. Use an image published after
it, set `APP_VERSION` in `.env` to that version, or build the image from
[MissionControlSquad/monorepo](https://github.com/MissionControlSquad/monorepo).

## What's here

| File | Purpose |
| --- | --- |
| [`docker-compose.yml`](docker-compose.yml) | PostgreSQL 16, Redis 7 and the AgentCenter app, hardened (dropped capabilities, read-only root, no-new-privileges) |
| [`docker-compose.prod.yml`](docker-compose.prod.yml) | Optional override that adds Traefik with automatic Let's Encrypt HTTPS |
| [`.env.example`](.env.example) | Every configuration variable, with how to generate each secret |
| [`SETUP.md`](SETUP.md) | Full guide: prerequisites, setup, HTTPS, upgrades, backups, troubleshooting |

## Quick start

```bash
git clone https://github.com/MissionControlSquad/agentcenter-self-hosting.git
cd agentcenter-self-hosting

cp .env.example .env
# Replace every placeholder secret with a random value:
sed -i "s|^PG_PASS=.*|PG_PASS=$(openssl rand -hex 16)|" .env
sed -i "s|^REDIS_PASS=.*|REDIS_PASS=$(openssl rand -hex 16)|" .env
sed -i "s|^AUTH_SECRET=.*|AUTH_SECRET=$(openssl rand -hex 32)|" .env
sed -i "s|^CRON_SECRET=.*|CRON_SECRET=$(openssl rand -hex 32)|" .env

docker compose up -d
```

On macOS, use `sed -i ''` instead of `sed -i`. For HTTPS, backups and the full variable reference,
see [SETUP.md](SETUP.md).

Never commit your `.env` file. It is listed in [`.gitignore`](.gitignore).

## Getting help

- Questions: [Discussions](https://github.com/MissionControlSquad/agentcenter-self-hosting/discussions)
- Bugs: [Issues](https://github.com/MissionControlSquad/agentcenter-self-hosting/issues/new/choose)
- Security: [report privately](https://github.com/MissionControlSquad/agentcenter-self-hosting/security/advisories/new), never in a public issue. See [SECURITY.md](SECURITY.md).

## Contributing

Pull requests are welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) and the
[Code of Conduct](CODE_OF_CONDUCT.md) first.

## License

[MIT](LICENSE) © 2026 Jagodana LLC
