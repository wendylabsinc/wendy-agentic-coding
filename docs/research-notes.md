# Research Notes

Checked on 2026-04-30 and refreshed on 2026-05-01.

## Codex

OpenAI's Codex plugin docs describe plugins as bundles of skills, app integrations, and MCP servers. The required plugin entry point is `.codex-plugin/plugin.json`. Optional bundled content such as `skills/`, `.app.json`, `.mcp.json`, and `assets/` should live at the plugin root, not inside `.codex-plugin/`.

Codex can read marketplace files from `.agents/plugins/marketplace.json` in a repository, from a Claude-style `.claude-plugin/marketplace.json`, and from a personal marketplace. The local Codex marketplace in this repo points at `./plugins/wendy-agentic-coding`.

Codex supports `codex plugin marketplace add owner/repo` for tracking a marketplace source. Plugin installation still happens through the plugin browser; installing the plugin makes skills available, but existing approval settings still apply to shell commands.

## Claude Code

Claude Code plugins use `.claude-plugin/plugin.json` for the plugin manifest. Plugin components such as `skills/`, `commands/`, `agents/`, `hooks/`, `.mcp.json`, `.lsp.json`, `monitors/`, `bin/`, and `settings.json` live at the plugin root.

Claude Code marketplace files live at `.claude-plugin/marketplace.json`. Marketplace entries can point to local plugin directories, which makes this repository installable as a Claude marketplace while keeping a single shared plugin payload.

Claude Code can install from CLI with `claude plugin marketplace add <source>` and `claude plugin install <plugin>@<marketplace>`. Claude plugins can include hooks that run commands on lifecycle events, but OS-level Wendy CLI installation is better left as an explicit skill/command action rather than a silent install hook.

## Compatibility Choice

This repo keeps one plugin payload under `plugins/wendy-agentic-coding/` and includes both manifests:

- `.codex-plugin/plugin.json` for Codex.
- `.claude-plugin/plugin.json` for Claude Code.

The reusable operational content lives in `skills/` because both tools understand skill-style instruction bundles. Claude-specific conveniences are additive: `commands/` for slash commands and `agents/` for a runtime debugger. Local validation accepted the agent when the manifest pointed directly at `["./agents/wendy-runtime-debugger.md"]`; the directory string form failed validation in the local Claude CLI despite the docs describing string paths.

## Hermes Agent

Checked on 2026-09-11 against `wendy` CLI `2026.08.18-022337` and Hermes Agent v0.21.0.

Hermes Agent (Nous Research) is an agent framework in the same category as Claude Code and Codex: terminal-based, tool-calling, MCP-capable, with its own marketplace of skills. Its extensibility surfaces relevant here are stdio MCP servers (`hermes mcp add`) and a directory-based skill loader (each skill is a folder with a `SKILL.md`, discovered from `~/.hermes/profiles/<profile>/skills/`).

The Wendy MCP server integrates without any CLI changes. `wendy mcp serve` is a standard stdio MCP server (JSON-RPC `initialize` and `tools/call` verified at the protocol level, 34 tools), and Hermes registers it directly:

```bash
hermes mcp add wendy --command wendy --args "mcp" "serve"
```

Verified end-to-end: registration connects (34 tools discovered), `hermes mcp test wendy` connects, and an in-session agent call to `wendy_status` / `device_list` returned structured results.

The gap is only in discoverability: `wendy mcp setup` has no Hermes target. Its binary installs MCP config for Claude Code and Claude Desktop, a skills digest for Codex and OpenCode (`wendy-skills.md`), but contains no Hermes path — so Hermes users must register manually. The generated skills digest also does not fit Hermes' layout: Hermes wants one `SKILL.md` per directory, whereas the Codex/OpenCode artifact is a single flat markdown file. The repository skills are already in the per-directory layout Hermes expects, so they can be symlinked into a profile's skills directory directly. Setup steps are documented in `plugins/wendy-agentic-coding/skills/wendy-hermes-setup/SKILL.md`.

A separate observation: `wendy mcp setup` installs the skills digest for OpenCode but does not write an MCP server entry into OpenCode's config (`mcp` block in `opencode.json`), despite OpenCode supporting local stdio servers there. Whether that asymmetry is intentional is unknown; it was noticed while checking which config files the setup writes.

No Hermes-side plugin artifacts (`plugin.yaml` + `__init__.py` registering tools/hooks/commands) were added. Hermes does support a `PluginManager` plugin system, but the MCP server plus markdown skills already cover the integration, and third-party-product plugins are out of scope for tool vendor repos by Hermes convention.

