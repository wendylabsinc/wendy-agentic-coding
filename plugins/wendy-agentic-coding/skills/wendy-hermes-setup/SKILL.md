---
name: wendy-hermes-setup
description: "Use when configuring the Wendy MCP server for Hermes Agent (Nous Research) with `hermes mcp add`, verifying device tools inside Hermes sessions, or installing wendy skills into a Hermes profile's skills directory."
---

# Wendy MCP Setup for Hermes Agent

Use this when the user runs Hermes Agent (https://github.com/NousResearch/hermes-agent) and wants access to Wendy device tools. `wendy mcp setup` does not configure Hermes, so registration is a one-line manual step — the MCP server itself (`wendy mcp serve`) is a standard stdio MCP server and needs no Hermes-side changes beyond registration.

## Preconditions

Verify the CLI first (same floor as other MCP hosts; `wendy mcp` and `wendy device hardware` need `2026.04.30-211221` or newer):

```bash
wendy --version
```

If a device should be connected at server startup, set a default device before registering:

```bash
wendy discover --json --timeout 5s
wendy device set-default <hostname>
```

## Register the MCP server

Register the stdio server with Hermes:

```bash
hermes mcp add wendy --command wendy --args "mcp" "serve"
```

Hermes probes the server during registration and lists the discovered tools. Confirm the entry landed and is enabled:

```bash
hermes mcp list
```

If the user wants the server pinned to one device, re-register with the device argument instead of editing config by hand:

```bash
hermes mcp add wendy --command wendy --args "mcp" "serve" "--device" "<hostname>"
```

On a fresh Hermes install the profile config is at `~/.hermes/profiles/<profile>/config.yaml`; the `hermes mcp add` command above writes the server there automatically. Do not hand-edit config unless the CLI version predates `hermes mcp add` — in that case add an `mcp_servers.wendy` entry with `command: wendy` and `args: ["mcp", "serve"]`, then start a new session.

## Verify

Connection test (launches the server over stdio and lists tools):

```bash
hermes mcp test wendy
```

Following registration, start a new Hermes session so the tool list is picked up. Inside a session, `wendy_status` reports connection state in plain English and `device_list` enumerates devices; both are safe read-only checks before running any device operation through [wendy-device-ops].

If `hermes mcp test wendy` fails, isolate the layer: run `wendy --version` (CLI missing or stale?), then `wendy discover --json` (device reachable?). The MCP server only proxies to a connected device; it cannot make an offline device reachable.

## Skills

`wendy mcp setup` writes a generated skills digest (named `wendy-skills.md`) for Codex and OpenCode, but Hermes discovers skills from per-directory `SKILL.md` files instead, searching `~/.hermes/skills/` (shared across all profiles) and `~/.hermes/profiles/<profile>/skills/` (per profile). This repository's skills (for example `wendy-app-lifecycle`, `wendy-device-ops`, `wendy-entitlements`) are already in that layout, so to make them discoverable to Hermes, run from the repository root and link all of them at once. Installing into `~/.hermes/skills/` makes them visible to every profile (a per-profile install uses `~/.hermes/profiles/<profile>/skills/` instead):

```bash
REPO_TOP=$(git rev-parse --show-toplevel)
mkdir -p ~/.hermes/skills
for dir in "$REPO_TOP"/plugins/wendy-agentic-coding/skills/*/; do
  ln -sfn "$dir" ~/.hermes/skills/"$(basename "$dir")"
done
```

`git rev-parse --show-toplevel` keeps the loop working from any subdirectory of the repository. Restart the session after adding skills — Hermes builds the skill index at session start.

## Safety

MCP exposes operational device tools to the assistant host. Do not configure it silently. Tell the user which config file was changed when a command reports it, and do not copy tokens, WiFi passwords, or device-specific secrets into reusable docs or issue text.
