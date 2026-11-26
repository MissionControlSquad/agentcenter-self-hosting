# Contributing

Thanks for helping. This repository holds deployment configuration, so small, well-tested changes
are the easiest to review and merge.

## Ground rules

- Be kind. Everyone taking part agrees to the [Code of Conduct](CODE_OF_CONDUCT.md).
- Security problems go through [private reporting](SECURITY.md), never a public issue or PR.
- For anything bigger than a fix, open a [discussion](https://github.com/MissionControlSquad/agentcenter-self-hosting/discussions)
  or an issue first, so the approach is agreed before you spend time on it.

## Making a change

1. Fork the repository and create a branch from `main` (for example `fix/redis-healthcheck`).
2. Make the change. Keep the existing container hardening (`cap_drop`, `read_only`,
   `no-new-privileges`, `pids_limit`) unless you explain in the PR why a service needs more.
3. Validate both compose files with a throwaway environment:

   ```bash
   cp .env.example .env
   docker compose config --quiet
   docker compose -f docker-compose.yml -f docker-compose.prod.yml config --quiet
   ```

4. If you add or rename a variable, update `.env.example` and the reference table in `SETUP.md`.
5. Commit using [Conventional Commits](https://www.conventionalcommits.org/) (`fix:`, `feat:`,
   `docs:`, `chore:`), then open a pull request against `main` and fill in the template.

## Never commit secrets

`.env` is git-ignored. Example values must be obvious placeholders (`change-me-…`). Secret
scanning with push protection is enabled on this repository and will block pushes that contain
recognised credentials.

## Review and merge

Every change lands through a pull request. The CI checks must pass and a code owner reviews it.
Pull requests are squash-merged, so the PR title becomes the commit message; write it as a
Conventional Commit.

## License

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE).
