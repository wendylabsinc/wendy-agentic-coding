---
name: wendy-project-setup
description: Use when creating, validating, or repairing a Wendy project scaffold, adapting Wendy templates, `wendy.json`, JSON schema setup, or project entitlements with `wendy init`, `wendy json`, or `wendy project entitlements`.
---

# Wendy Project Setup Workflow

Use this before app lifecycle work when the repository does not yet have a reliable Wendy project shape.

For new app creation, prefer the `wendy-template-app` workflow first. Search the local `../templates` repo or `wendylabsinc/templates` for a close match before writing a scaffold from scratch.

## Non-interactive project creation

Prefer explicit flags so coding agents avoid template, entitlement, and assistant prompts.

Minimal WendyOS app:

```bash
wendy init --app-id <app-id> --target wendyos --language python --no-extra-entitlements --assistant skip
```

Template scaffold:

```bash
wendy init --app-id <app-id> --target wendyos --language rust --template simple-api --var PORT=8080 --assistant skip --git-init no
```

Template init can still prompt if the selected template has required variables that are not supplied with `--var`. Pass every required template variable, and set `--git-init yes` or `--git-init no`, when the goal is a fully non-interactive agent run.

If the user names an archetype such as `realsense-camera`, `camera-feed`, `camera-feed-yolo`, `voice-assistant`, `audio`, `fullstack`, or `simple-api`, inspect the matching template's `template.json`, `wendy.json`, `Dockerfile`, and source files before creating the destination app.

Project with entitlements:

```bash
wendy init \
  --app-id <app-id> \
  --target wendyos \
  --language python \
  --entitlement camera,audio,persist \
  --persist-name <app-id>-data \
  --persist-path /data \
  --assistant skip
```

Hardware app with GPIO and I2C:

```bash
wendy init \
  --app-id edge-sensors \
  --target wendyos \
  --language swift \
  --entitlement gpio,i2c \
  --gpio-pins 17,27,22 \
  --i2c-device i2c-1 \
  --assistant skip
```

Useful `wendy init` flags:

- `--app-id`: application ID written to `wendy.json`.
- `--target`: `wendyos` or `wendy-lite`.
- `--language`: `python`, `swift`, `rust`, `node`, or `cpp`.
- `--template`: template name; a bare `--template` opens a picker, so avoid bare usage in agent workflows.
- `--branch`: templates repo branch.
- `--var KEY=VALUE`: repeatable template variable override.
- `--entitlement`: repeatable or comma-separated entitlement list.
- `--all-entitlements`: enable every entitlement, with required field flags supplied.
- `--no-extra-entitlements`: skip extra entitlement prompts and keep defaults.
- `--gpio-pins`, `--i2c-device`, `--persist-name`, `--persist-path`: required field values for those entitlements. Use `i2c-1`, not `/dev/i2c-1`, for `--i2c-device`.
- `--assistant`: `claude`, `codex`, or `skip`; use `skip` unless the user explicitly wants the assistant launched.
- `--install-claude-skills`: only use with `--assistant claude` when requested.
- `--git-init`: `yes` or `no`.

## Validate `wendy.json`

Validate the current directory:

```bash
wendy json validate
```

Validate a specific file or directory:

```bash
wendy json validate ./wendy.json
wendy json validate ./path/to/app
```

Print the schema for editor setup:

```bash
wendy json schema > wendy.schema.json
```

`wendy json validate` reports deprecated or suspicious config as warnings after structural validation. Treat warnings as actionable, especially `video` entitlement warnings where new configs should use `camera`.

## Project entitlement commands

List configured entitlements:

```bash
wendy --json project entitlements list
```

List available entitlement types:

```bash
wendy --json project entitlements list --show-all
```

Add or remove a simple entitlement by name:

```bash
wendy project entitlements add camera
wendy project entitlements remove camera
```

Important: `project entitlements add` prompts for required fields for `persist`, `i2c`, and `gpio`. In an agent workflow, prefer editing `wendy.json` directly for those entitlements, then run `wendy json validate`.

For I2C, use the device name without `/dev/`, for example `i2c-1`. The runtime entitlement path prefixes `/dev/` when mounting the device.

## `wendy.json` top-level fields

Beyond `appId` and `entitlements`, the schema (`wendy json schema`) accepts:

- `platform`: `linux`, `wendyos`, `darwin` (optionally `/<arch>`), or `wendy-lite`. Docs examples use `linux`; `wendyos` is equivalent for container targets. `darwin` apps are native (non-containerized) and do not use entitlements.
- `version`, `language` (selects the build toolchain: `swift`, `python`, `rust`, `node`, `cpp`).
- `debug` (bool, default false): injects debug tooling via the `WENDY_DEBUG` build arg.
- `brewfile` (string, `darwin` only): Homebrew Bundle manifest, relative path, no `..`.
- `readiness`: `{ "tcpSocket": { "port": <1-65535> }, "timeoutSeconds": 30 }`. Only `tcpSocket` is supported (no `httpGet`/`exec`). Single-container apps only.
- `hooks.postStart`: `{ "cli": "<runs on dev machine>", "agent": "<runs on device>" }`. Commands run directly, not through a shell. Single-container apps only.
- `resources`: `{ "memory": "512Mi", "cpus": "1.5", "pids": 256 }`. `memory` suffixes `Ki|Mi|Gi|Ti|K|M|G|T`; `pids` default 4096.
- `python`: `{ "sourceRoot": "...", "container": { "sourceRoot": "..." } }`.

## Multi-service apps (`services` map)

For an app that runs more than one container from one `wendy.json`, use a `services` map instead of a single top-level context. Each service builds from its own `context` directory (one Dockerfile per service):

```json
{
  "appId": "com.example.stack",
  "platform": "linux",
  "version": "1.0.0",
  "services": {
    "db": { "context": "./db" },
    "api": {
      "context": "./api",
      "dependsOn": ["db"],
      "entitlements": [ { "type": "network", "mode": "host" } ]
    },
    "frontend": { "context": "./frontend", "dependsOn": ["api"] }
  }
}
```

- `context` is required per service, relative, and must not contain `..`. Each service may carry its own `entitlements`, `dependsOn`, `frameworks`, and `resources`.
- `dependsOn` sets creation order: `wendy run` builds images in parallel, deploys containers in topological order, and stops them in reverse order. Cycles are a validation error.
- Multi-service apps are Linux/WendyOS only (a headless Mac target is rejected). Readiness probes and `postStart` hooks are single-container only.
- Prefer a `persist` entitlement over host bind mounts for per-service storage; dev-machine paths do not exist on the device.
- A `docker-compose.yml` with no `services` map makes `wendy.json` a companion that layers Wendy-only settings (GPU, network, etc.) onto Compose-defined services. Hardware entitlements are not inferred from Compose.

`isolation` (top-level) and `frameworks.ros2` (top-level and per-service) apply to multi-service apps; see [wendy-entitlements] for `isolation` modes and [wendy-ros2] for ROS 2 apps. Validate any multi-service or ROS 2 config with `wendy json validate` before building.

## Setup checklist

- Prefer an existing Wendy template before creating an app from scratch.
- Use `wendy init` for a new project scaffold.
- Use the `wendy-entitlements` skill when choosing capability details.
- Keep `appId` stable once a device has deployed the app.
- Keep JSON examples valid; `wendy.json` cannot contain comments.
- Validate with `wendy json validate` before `wendy build` or `wendy run`.
