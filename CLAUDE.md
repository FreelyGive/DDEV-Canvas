# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **DDEV add-on** (not an application). It layers Canvas Storybook AI tooling onto an *existing* DDEV
project: a separate Storybook service, plus Claude Code, glab, and agent-browser inside the web container.
There is no application source, no build, and no test suite here — the repo is a set of DDEV config
fragments that get copied into a target project's `.ddev/` directory by `ddev add-on get`.

`install.yaml` is the manifest: its `project_files` list is the authoritative set of files this add-on
installs. **Any new file the add-on must ship has to be added there**, or DDEV will not copy it.

## How files map into the target project

`ddev add-on get` copies `project_files` into the consuming project's `.ddev/` directory, preserving
relative paths. So:
- `config.canvas.yaml` → `.ddev/config.canvas.yaml` (merged into DDEV config; `config.*.yaml` files are auto-merged)
- `commands/web/claude` → `.ddev/commands/web/claude` → becomes the `ddev claude` host command
- `web-build/Dockerfile.canvas` → `.ddev/web-build/Dockerfile.canvas` → appended to the web container image build
- `traefik/config/canvas-storybook.yaml` → custom Traefik router for the Storybook subdomain

The `#ddev-generated` marker at the top of every file tells DDEV the file is add-on-managed (lets it be
cleanly removed/updated). Keep that marker on files that ship.

## Architecture

**Two moving parts:**

1. **Storybook service** (`docker-compose.canvas-storybook.yaml`) — a standalone `node:20` container,
   `working_dir: /var/www/html/storybook`, running `npm install && npm run dev --host=0.0.0.0` on port 6006.
   The host `node_modules` is **shadowed by a named volume** (`canvas-storybook-node-modules`) so
   platform-specific binaries (esbuild, etc.) are compiled for Linux, not the host OS. Routing to it is
   done by the Traefik config + the `storybook.<project>` additional hostname (for the TLS cert).

2. **Web-container tooling** (`web-build/Dockerfile.canvas`) — installs Claude Code (`npm i -g
   @anthropic-ai/claude-code`), glab, agent-browser, and Chromium into the existing web container.
   Chromium is used instead of Chrome-for-Testing because there are no ARM64 Linux Chrome builds.

**Claude Code config persistence** (`config.canvas.yaml` post-start hooks): `~/.claude.json` and
`~/.claude` inside the container are symlinked to `.ddev/claude-code/` on the host, so config survives
container rebuilds. Those targets are gitignored (`claude-code/.gitignore`) — they are per-developer state,
never committed.

## Design constraint — non-invasive config

`config.canvas.yaml` must **only add** to DDEV config. It deliberately does NOT set `type`, `docroot`,
`php_version`, `webserver_type`, `database`, or `omit_containers`, so it can be installed alongside any
existing project without clobbering its setup. Preserve this when editing — only use additive keys
(`hooks`, `web_environment`, etc.).

## Things that need manual editing after install

`storybook.my-project` is a literal placeholder in two files — it must be replaced with the real DDEV
project name by the consumer (or by the Canvas installer script):
- `config.canvas-storybook-hostname.yaml` (`additional_hostnames`)
- `traefik/config/canvas-storybook.yaml` (the `HostRegexp` rules and the upstream `ddev-my-project-storybook` URL)

## Working on the add-on

There is nothing to build or lint. To validate a change, install it into a real DDEV project and restart:

```bash
ddev add-on get /path/to/DDEV-Canvas   # install local working copy
ddev restart                           # rebuild containers, run post-start hooks
ddev describe                          # confirm the storybook URL / hostname
ddev logs -s storybook                 # debug the Storybook service
```

Host commands exposed to consumers: `ddev claude`, `ddev glab`, `ddev playwright-cli`.
