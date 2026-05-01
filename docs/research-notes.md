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
