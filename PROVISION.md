# Provision an OpenAction agent workspace environment

Provision this Ubuntu VPS as a agent workspace environment. Install the packages below, configure them for the 
current user, and verify that every command is available. Use `sudo` for system changes and make the setup safe 
to run again.

## Required packages

- Ubuntu packages: `bubblewrap`, `ca-certificates`, `curl`, `git`, `gnupg`, `lsb-release`, `sqlite3`, `unzip`, `make`.
- GitHub CLI (`gh`): latest stable version from GitHub CLI's official APT repository.
- ForgeJo CLI (`fj`): latest stable version
- Docker Engine: latest stable major version, with Buildx and Docker Compose plugins. Enable and start the service.
- NVM: `0.40`.
- Node.js: major version `22`, configured as the NVM default, and major version `24` usable as well, and `corepack`
  enabled in both.
- PHP: major/minor version `8.4`, from the native `resolute` suite at `https://packages.sury.org/php/`.
- Symfony CLI: latest stable version.
- Composer: latest stable version, with the installer SHA-384 checksum verified before installation.
- Playwright CLI: latest stable version from `@playwright/cli`, with Chromium's headless shell and Linux dependencies.
- RTK: latest stable version from its official installer.

## Required PHP extensions

Install the PHP 8.4 packages for:

- APCu
- AMQP
- BCMath
- cURL
- GD
- GMP
- Intl
- Mbstring
- OPcache
- PostgreSQL
- Readline
- Redis
- SQLite3
- XML
- ZIP

Also install the PHP 8.4 CLI and common packages, and configure PHP 8.4 as the default `php` executable.

Install Sury's signed archive keyring and configure its repository with the Ubuntu codename reported by the host. On
Ubuntu 26.04, this must resolve to the repository's native `resolute` suite. Do not use `ppa:ondrej/php`, Ubuntu 24.04
`noble` packages, cross-release APT pinning, or compatibility libraries from another Ubuntu release. If the native suite
is unavailable, stop and report the error instead of falling back to packages from a different Ubuntu release.

Install GitHub CLI from its official APT repository at `https://cli.github.com/packages`. Configure the repository's
signed archive keyring and the `stable main` source for the host architecture, then install `gh` after refreshing APT.
Make repository and keyring setup safe to run again.

## Required configuration

- Configure Playwright for headless Chromium with the Chromium sandbox disabled.
- Set `PLAYWRIGHT_MCP_CONFIG` to the Playwright CLI configuration file in the user's shell environment.
- Add `~/.local/bin` to the user's `PATH`.
- Configure Git as `OpenAction Agent <agent@openaction.eu>`.
- Add a non-root user to the `docker` group when necessary.

## Verification

Verify Docker, Docker Compose, Node.js, PHP, Symfony CLI, Composer, Playwright CLI, RTK, GitHub CLI (`gh`), and Git
identity.
Confirm that PHP 8.4 loads the APCu, AMQP, and Redis extensions and that Docker is enabled and running.
Confirm with `apt-cache policy` that PHP 8.4 comes from Sury's `resolute` suite and no `noble` source or pin remains.

Report the installed versions and any failed step. Do not include secrets in the report.
