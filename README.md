# Wendy

Shared Codex and Claude Code plugin content for developers building with Wendy.

It teaches agents how the Wendy CLI, `wendy-agent`, WendyOS, project setup, `wendy.json`, entitlements, device operations, MCP setup, app lifecycle commands, Wendy Data, and device debugging actually work.

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
curl -fsSL https://install.wendy.dev/cli.sh | bash
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
- Use `audio`, `camera`, `display`, `bluetooth`, `usb`, `serial`, `i2c`, `gpio`, `spi`, and `input` only when the app actually needs those host devices.
- For `i2c`, include `device`, for example `{ "type": "i2c", "device": "i2c-1" }` (bare `i2c-N`, not a `/dev/` path).
- For `serial`, `device` is a bare USB tty node such as `ttyACM0`/`ttyUSB0`.
- For `gpio`, optional `pins` are documentation/validation; runtime access is chip-level.
- Use `mcp` (requires `port`) to expose a container as an MCP server; use `admin` only for fully-trusted first-party apps (unauthenticated local device control).
- For multi-service and ROS 2 apps, use the top-level `isolation` and `frameworks.ros2` keys; see the `wendy-robot-deploy` skill.

## What It Includes

Skills:

- `wendy-onboarding`: guide a first-time user to a physical device or a simulator.
- `wendy-install`: install, update, and verify the Wendy CLI, MCP server, and plugin.
- `wendy-device-install`: install WendyOS on a new Raspberry Pi or Jetson and verify first boot.
- `wendy-mcp-setup`: configure and explain the Wendy MCP server for AI assistants.
- `wendy-template-app`: create a Wendy app from the closest template in `wendylabsinc/templates`.
- `wendy-project-setup`: initialize projects, validate `wendy.json`, and manage project entitlements.
- `wendy-entitlements`: choose and validate `wendy.json` entitlements.
- `wendy-device-ops`: discover devices, inspect hardware and software, configure WiFi, and update the agent.
- `wendy-app-lifecycle`: build, run, detach, stream logs, manage apps, and clean up volumes.
- `wendy-robot-deploy`: deploy ROS 2 and robot apps, including Unitree Go2/G1 and simulators.
- `wendy-data`: Wendy Data campaigns, episodes, and application recording streams.
- `wendy-device-debug`: diagnose unreachable devices, failed deploys, and unhealthy apps.
- `wendy`: a single overview skill that links to the others.

Claude Code commands: `wendy-apps`, `wendy-build`, `wendy-data`,
`wendy-device-debug`, `wendy-device-install`, `wendy-device-ops`,
`wendy-entitlements`, `wendy-install`, `wendy-logs`, `wendy-mcp`,
`wendy-project`, `wendy-robot-deploy`, `wendy-run`, and `wendy-template`, each
under `/wendy-agentic-coding:`.

The Claude Code plugin also starts the Wendy MCP server (`wendy mcp serve`) and
runs a SessionStart hook that reports the installed CLI version in Wendy
projects. See the [plugin README](plugins/wendy-agentic-coding/README.md).

## Repository Layout

This repository is a marketplace for both Codex and Claude Code:

- Codex marketplace: `.agents/plugins/marketplace.json`
- Claude Code marketplace: `.claude-plugin/marketplace.json`
- Plugin: `plugins/wendy-agentic-coding/`

The plugin is published from
[`wendylabsinc/wendyos`](https://github.com/wendylabsinc/wendyos/tree/main/plugins/wendy-agentic-coding).
Make changes there; this repository is a copy.

For local development:

```bash
claude --plugin-dir ./plugins/wendy-agentic-coding
```

## Sources Checked

The structure follows current plugin guidance from:

- OpenAI Codex plugin docs: <https://developers.openai.com/codex/plugins> and <https://developers.openai.com/codex/plugins/build>
- Claude Code plugin docs: <https://code.claude.com/docs/en/plugins> and <https://code.claude.com/docs/en/plugins-reference>
