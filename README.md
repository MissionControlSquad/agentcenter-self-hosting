# AgentCenter Self-Hosting

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Validate](https://github.com/MissionControlSquad/agentcenter-self-hosting/actions/workflows/validate.yml/badge.svg)](https://github.com/MissionControlSquad/agentcenter-self-hosting/actions/workflows/validate.yml)

Docker Compose files and a setup guide for running AgentCenter, an AI agent management dashboard,
on your own server.

## Project status

The hosted AgentCenter service at agentcenter.cloud has been shut down, and this repository is now
published under the [MIT License](LICENSE). It is provided as-is and maintained on a best-effort
basis by the community.

One thing to know before you start: the published app image
(`ghcr.io/missioncontrolsquad/agentcenter`) asks for a license key on first start and validates it
against the hosted license server. That server is gone, so a new installation cannot be activated,
and an existing one stops working a few days after its cached license token expires. The app's
source lives in [MissionControlSquad/monorepo](https://github.com/MissionControlSquad/monorepo);
follow that repository for any change to the license check.

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
