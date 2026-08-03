# OpenAction agent workspace environment

This repository contains the OpenAction Agent workspace environment (provisioning, skills, sub agents, ...). 

## Supported system

- Ubuntu 26.04 LTS (Resolute)
- A root shell, or a regular login user with `sudo` access
- An interactive terminal and outbound internet access

## Provision the VPS

Clone the repository as the user that will run Codex, then run:

```bash
./bin/provision.sh
```

## Finish the setup

Reconnect over SSH (or start a new login shell) so the MetaMCP environment variable and, for non-root users, 
Docker group membership take effect. 

Then authenticate Codex:

```bash
codex login
codex mcp add sentry --url https://mcp.sentry.dev/mcp
codex mcp add linear --url https://mcp.linear.app/mcp
codex mcp add github --url https://api.githubcopilot.com/mcp/ --bearer-token-env-var GITHUB_PAT_TOKEN
```

And add GITHUB_PAT_TOKEN env var (with token) to your .bashrc file.

## Configure OpenAction instance MCPs

Add one clearly named server for each instance. For example, Place Publique:

```bash
codex mcp add oa-placepublique --url https://console.mobilisation-place-publique.eu/mcp
codex mcp login oa-placepublique
```

Suggested local names are `placepublique` (`pp`), `ecologistes` (`ecolos`),
`lapres`, and `europe`. Names are configurable; run `codex mcp list` to check
the active configuration.
