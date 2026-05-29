# DDEV Canvas Addon

Adds [Canvas Storybook AI](https://canvas.drupalstarforge.ai) tooling to an existing DDEV project.

## What this installs

- **Storybook service** — runs Storybook in a separate Node 20 container at `https://storybook.<project>.ddev.site`
- **Claude Code** — installs Claude Code inside the web container with isolated config
- **glab CLI** — GitLab CLI inside the container
- **Playwright CLI** — browser automation for component migration and testing
- **Canvas sync hooks** — keeps local Canvas components in sync on container start

## Requirements

- [DDEV](https://ddev.readthedocs.io/) v1.23+
- Docker
- An existing project with a `package.json` that includes Storybook (or use the [Canvas Storybook AI installer](https://canvas.drupalstarforge.ai) to scaffold one)

## Installation

```bash
ddev add-on get FreelyGive/DDEV-Canvas
ddev restart
```

After restart, update `storybook.my-project` in `.ddev/config.canvas-storybook-hostname.yaml` to match your actual DDEV project name, then run `ddev restart` again.

Or use the Canvas installer which handles this automatically:

```bash
curl -fsSL https://canvas.drupalstarforge.ai/install.sh | bash
```

## Usage

```bash
ddev claude          # Launch Claude Code inside the container
ddev glab            # GitLab CLI
ddev playwright-cli  # Playwright browser automation
```

Storybook starts automatically when DDEV starts. Access it at:
```
https://storybook.<your-project>.ddev.site
```

## Configuration

Copy `.env.example` to `.env` and fill in your Canvas/Drupal credentials:

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `CANVAS_SITE_URL` | Your Acquia/Drupal CMS API URL |
| `CANVAS_CLIENT_ID` | OAuth client machine name |
| `CANVAS_CLIENT_SECRET` | OAuth client secret |
| `CANVAS_JSONAPI_PREFIX` | JSON:API prefix (`api` for Acquia, `jsonapi` for Drupal Canvas) |
