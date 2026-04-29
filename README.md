# Wendy Agentic Coding

Shared plugin content for agents working on Wendy Labs repositories, especially `wendy-agent`, WendyOS, sample apps, and device workflows.

This repository is structured as a plugin marketplace for both Codex and Claude Code:

- Codex marketplace: `.agents/plugins/marketplace.json`
- Claude Code marketplace: `.claude-plugin/marketplace.json`
- Shared plugin payload: `plugins/wendy-agentic-coding/`
- Shared skills: `plugins/wendy-agentic-coding/skills/`
- Claude-only commands and agent definitions: `plugins/wendy-agentic-coding/commands/` and `plugins/wendy-agentic-coding/agents/`

## Install in Codex

Clone or use this repository from a Wendy workspace, then open the Codex plugin browser:

```bash
codex
/plugins
```

Codex reads repo marketplaces from `.agents/plugins/marketplace.json`. Install `wendy-agentic-coding`, start a new thread, and ask Codex to use Wendy Agentic Coding.

## Install in Claude Code

Add this repository as a marketplace, then install the plugin:

```text
/plugin marketplace add wendylabsinc/wendy-agentic-coding
/plugin install wendy-agentic-coding@wendy-agentic-coding
/reload-plugins
```

For local development:

```bash
claude --plugin-dir ./plugins/wendy-agentic-coding
```

Claude Code exposes namespaced commands such as `/wendy-agentic-coding:wendy-orient`, `/wendy-agentic-coding:wendy-device-debug`, and `/wendy-agentic-coding:wendy-pr`.

## What It Teaches

The plugin captures the current Wendy CLI and agent architecture:

- `wendy` CLI is a Cobra app under `wendy-agent/go/cmd/wendy`.
- CLI command wiring lives in `wendy-agent/go/internal/cli/commands`.
- gRPC client setup lives in `wendy-agent/go/internal/cli/grpcclient`.
- Build and run providers live in `wendy-agent/go/internal/cli/providers`.
- `wendy-agent` starts gRPC services from `wendy-agent/go/cmd/wendy-agent`.
- Agent services live under `wendy-agent/go/internal/agent/services`.
- Container runtime integration uses containerd code under `wendy-agent/go/internal/agent/containerd`.
- Runtime entitlements and OCI spec generation live under `wendy-agent/go/internal/agent/oci`.

The skills emphasize reading the real code path, preserving dirty worktrees, verifying runtime wiring, using live device state when needed, and running targeted Go validation before publishing.

## Sources Checked

The structure follows current plugin guidance from:

- OpenAI Codex plugin docs: <https://developers.openai.com/codex/plugins> and <https://developers.openai.com/codex/plugins/build>
- Claude Code plugin docs: <https://code.claude.com/docs/en/plugins> and <https://code.claude.com/docs/en/plugins-reference>
