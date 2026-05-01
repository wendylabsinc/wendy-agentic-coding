# Wendy

Shared Codex and Claude Code plugin content for developers building with Wendy.

It teaches agents how the Wendy CLI, `wendy-agent`, WendyOS, project setup, `wendy.json`, entitlements, device operations, MCP setup, app lifecycle commands, device debugging, and PR workflows actually work.

License: Apache-2.0.

## Quick Start

Full Wendy agentic-coding setup has two parts:

1. Install this plugin into Codex or Claude Code.
2. Install or update the Wendy CLI, then verify device discovery.

### Codex

Add this repo as a Codex marketplace:

```bash
codex plugin marketplace add wendylabsinc/wendy-agentic-coding
```

Then open Codex, install the plugin from the plugin browser, and start a new thread:

```bash
codex
/plugins
```

Try:

```text
Use @Wendy to install Wendy CLI, verify my Wendy CLI version, discover my Wendy device, and create a wendy.json for this app.
```

Template-first app creation example:

```text
Use @Wendy to create a Wendy app in ./foo-bar from the realsense-camera template. Search local templates first; if source behavior is unclear, check wendylabsinc/templates before writing code.
```

### Claude Code

Add the marketplace and install the plugin:

```bash
claude plugin marketplace add wendylabsinc/wendy-agentic-coding
claude plugin install wendy-agentic-coding@wendy-agentic-coding
```

Restart Claude Code or run:

```text
/reload-plugins
```

Try:

```text
/wendy-agentic-coding:wendy-install
/wendy-agentic-coding:wendy-template
/wendy-agentic-coding:wendy-project
/wendy-agentic-coding:wendy-entitlements
/wendy-agentic-coding:wendy-device-ops
/wendy-agentic-coding:wendy-mcp
/wendy-agentic-coding:wendy-device-debug
```

## Wendy CLI Setup

The plugin does not silently install the Wendy CLI when you install the plugin. That is intentional: installing system tools should remain an explicit agent action so the developer can see and approve the command.

For the full plugin workflow, including Wendy MCP and hardware inspection, use Wendy CLI `2026.04.30-211221` or newer. Older CLI builds can still support basic install, discovery, build, and run flows, but may not include commands such as `wendy mcp` or `wendy device hardware`.

After the plugin is installed, ask Codex or Claude Code to set up Wendy. The agent should detect the OS, check whether `wendy` is already installed, and then use the right command:

macOS / Linux:

```bash
curl -fsSL https://install.wendy.sh/cli.sh | bash
```

Windows:

```powershell
winget install WendyLabs.Wendy --source winget
```

Verification:

```bash
wendy --version
wendy discover --json
```

If `wendy --version` is older than `2026.04.30-211221`, update Wendy before using the MCP or hardware skills. If a package manager has not caught up to the latest Wendy release, install the latest CLI release from <https://github.com/wendylabsinc/wendy-agent/releases>.

## First App Checklist

After the plugin and Wendy CLI are installed:

1. Confirm the CLI is available: `wendy --version`.
2. Confirm a device is reachable: `wendy discover --json`.
3. Ask the agent to create or validate `wendy.json` for the app.
4. Run the app in the background: `wendy run --yes --detach --device <hostname>`.
5. Stream logs or inspect state with the app lifecycle skill.

For new apps, ask the agent to search templates first. The plugin teaches agents to prefer the local `../templates` repo or `wendylabsinc/templates` before hand-writing a scaffold.

## `wendy.json` Entitlements

Use the `wendy-entitlements` skill when an app needs device access. It is based on the runtime switch in `wendy-agent/go/internal/agent/oci/entitlements.go`.

Common example:

```json
{
  "appId": "camera-assistant",
  "platform": "wendyos",
  "entitlements": [
    { "type": "network", "mode": "host" },
    { "type": "camera" },
    { "type": "audio" },
    { "type": "gpu" },
    { "type": "persist", "name": "camera-assistant-data", "path": "/data" }
  ],
  "readiness": {
    "tcpSocket": { "port": 8000 },
    "timeoutSeconds": 30
  }
}
```

Rules of thumb:

- Use `camera`, not `video`; `video` is a deprecated compatibility alias.
- Use `network` with `mode: "host"` for servers, WebRTC, callbacks, and debugger ports.
- Use `persist` for model caches, app state, databases, uploaded files, and generated assets.
- Use `gpu` only when the app is built for NVIDIA/Jetson acceleration and the device image has CDI set up.
- Use `audio`, `camera`, `bluetooth`, `usb`, `i2c`, `gpio`, `spi`, and `input` only when the app actually needs those host devices.
- For `i2c`, include `device`, for example `{ "type": "i2c", "device": "i2c-1" }`.
- For `gpio`, optional `pins` are documentation/validation; runtime access is chip-level.

## What It Includes

Skills:

- `wendy-codebase`: orient on Wendy CLI, agent, gRPC, runtime, and repo layout.
- `wendy-install`: install and verify Wendy CLI on macOS, Linux, or Windows.
- `wendy-template-app`: create or adapt Wendy apps from the closest template in `wendylabsinc/templates`.
- `wendy-project-setup`: initialize projects, validate `wendy.json`, print schema, and manage project entitlements.
- `wendy-entitlements`: choose and validate `wendy.json` entitlements.
- `wendy-device-ops`: discover/select devices, inspect version/hardware/WiFi state, update agents, and gather diagnostic evidence.
- `wendy-mcp-setup`: configure and explain the Wendy CLI MCP server for AI assistants.
- `wendy-app-lifecycle`: build, run, detach, stream logs, manage apps, and clean up volumes.
- `wendy-device-debug`: debug WendyOS and live-device runtime issues.
- `wendy-pr-workflow`: prepare, validate, publish, and clean up Wendy PRs.

Claude Code commands:

- `/wendy-agentic-coding:wendy-orient`
- `/wendy-agentic-coding:wendy-install`
- `/wendy-agentic-coding:wendy-template`
- `/wendy-agentic-coding:wendy-project`
- `/wendy-agentic-coding:wendy-entitlements`
- `/wendy-agentic-coding:wendy-device-ops`
- `/wendy-agentic-coding:wendy-mcp`
- `/wendy-agentic-coding:wendy-build`
- `/wendy-agentic-coding:wendy-run`
- `/wendy-agentic-coding:wendy-logs`
- `/wendy-agentic-coding:wendy-apps`
- `/wendy-agentic-coding:wendy-device-debug`
- `/wendy-agentic-coding:wendy-pr`

Claude Code agent:

- `wendy-agentic-coding:wendy-runtime-debugger`

## Repository Layout

This repository is a marketplace for both Codex and Claude Code:

- Codex marketplace: `.agents/plugins/marketplace.json`
- Claude Code marketplace: `.claude-plugin/marketplace.json`
- Shared plugin payload: `plugins/wendy-agentic-coding/`
- Shared skills: `plugins/wendy-agentic-coding/skills/`
- Claude-only command wrappers: `plugins/wendy-agentic-coding/commands/`
- Claude-only agent definition: `plugins/wendy-agentic-coding/agents/wendy-runtime-debugger.md`

For local development:

```bash
claude --plugin-dir ./plugins/wendy-agentic-coding
```

## Wendy Code Map

The plugin captures this current Wendy CLI and agent architecture:

- `wendy` CLI is a Cobra app under `wendy-agent/go/cmd/wendy`.
- CLI command wiring lives in `wendy-agent/go/internal/cli/commands`.
- gRPC client setup lives in `wendy-agent/go/internal/cli/grpcclient`.
- Build and run providers live in `wendy-agent/go/internal/cli/providers`.
- `wendy-agent` starts gRPC services from `wendy-agent/go/cmd/wendy-agent`.
- Agent services live under `wendy-agent/go/internal/agent/services`.
- Container runtime integration uses containerd code under `wendy-agent/go/internal/agent/containerd`.
- Runtime entitlements and OCI spec generation live under `wendy-agent/go/internal/agent/oci`.

The skills emphasize reading the real code path, preserving dirty worktrees, verifying runtime wiring, using live device state when needed, and running targeted Go validation before publishing.

## Known Gaps

This plugin is intentionally instruction-first. It does not bundle an app connector, LSP config, or automatic system installer. For MCP, it guides agents to the Wendy CLI's built-in `wendy mcp` commands.

Possible next additions:

- A safe Claude hook that only verifies `wendy --version` at session start and suggests setup if missing.
- A high-friction `wendy-os-image` skill for OS download/install/drive flashing, with explicit destructive-action confirmation rules.
- A `wendy-cloud` skill if Cloud run, auth, tunnels, or remote discovery become common agent workflows.
- More sample `wendy.json` recipes by app type: robotics, sensors, RealSense variants, and GPU inference.
- A Codex/Claude eval suite that checks whether the skills choose the right entitlements from app descriptions.

## Sources Checked

The structure follows current plugin guidance from:

- OpenAI Codex plugin docs: <https://developers.openai.com/codex/plugins> and <https://developers.openai.com/codex/plugins/build>
- Claude Code plugin docs: <https://code.claude.com/docs/en/plugins> and <https://code.claude.com/docs/en/plugins-reference>
