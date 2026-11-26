# Security Policy

## Project status

The hosted AgentCenter service at `agentcenter.cloud` has been shut down. This repository is
published under the [MIT License](LICENSE) and is provided as-is. There is no hosted
infrastructure, license server or customer data operated by the maintainers any more, so reports
about `*.agentcenter.cloud` hosts are out of scope.

## Supported versions

Security fixes are made on the default branch (`main`) only. Older commits, tags and container
images are not patched; please reproduce against the latest `main` before reporting.

| Version            | Supported |
| ------------------ | --------- |
| `main` (latest)    | Yes       |
| Anything older     | No        |

## Reporting a vulnerability

**Do not open a public issue, discussion or pull request for a security problem.**

Report it privately through GitHub's private vulnerability reporting:

1. Go to **[Security → Report a vulnerability](https://github.com/MissionControlSquad/agentcenter-self-hosting/security/advisories/new)**.
2. Describe the issue, the affected file(s) or component, and the impact you can demonstrate.
3. Include clear reproduction steps or a minimal proof of concept, and the commit you tested.

Only the reporter and the repository maintainers can see the report.

## What to expect

This project is maintained on a best-effort basis by volunteers. We aim to:

- acknowledge a report within **7 days**,
- share an initial assessment within **14 days**, and
- agree a disclosure date with you once a fix or mitigation is ready (by default, no later than
  **90 days** after the report).

Fixes are published as a [GitHub Security Advisory](https://github.com/MissionControlSquad/agentcenter-self-hosting/security/advisories), with a CVE
requested through GitHub where appropriate, and reporters are credited unless they prefer not to be.

## Scope

In scope: code and configuration in this repository, including workflows under `.github/`.

Out of scope:

- the decommissioned `agentcenter.cloud` service and its subdomains;
- findings in third-party dependencies that are already publicly known (Dependabot tracks these);
- reports that need a compromised host, physical access, or social engineering of maintainers;
- missing hardening headers or best-practice notes with no demonstrated impact.

## Safe harbor

We will not pursue or support legal action against anyone who researches and reports in good
faith under this policy: avoid privacy violations and data destruction, do not degrade other
people's systems, and give us reasonable time to respond before any disclosure.
