# Repository Instructions

This repository packages Wendy Labs agentic-coding workflows as installable Codex and Claude Code plugins.

Keep reusable instructions inside `plugins/wendy-agentic-coding/skills/` whenever possible. Use Claude-only `commands/` and `agents/` as thin wrappers around the shared skills so Codex and Claude stay aligned.

Do not commit device credentials, local hostnames, tokens, or private machine paths beyond documented Wendy workspace examples. If device access is needed, refer agents to the local repo's `AGENTS.md` or the user's current instructions.

When changing plugin manifests, keep both:

- `plugins/wendy-agentic-coding/.codex-plugin/plugin.json`
- `plugins/wendy-agentic-coding/.claude-plugin/plugin.json`

When changing marketplace metadata, keep both:

- `.agents/plugins/marketplace.json`
- `.claude-plugin/marketplace.json`
