# config

Sanitized **templates** for Claude Code configuration that lives *outside* the plugin —
your top-level user `settings.json` and your MCP server config. These are not part of the
plugin (Claude Code doesn't bundle them), but they're kept here so they're versioned too.

> **This repo is public.** Only `*.example.json` templates belong here. Never commit your
> real `settings.json` or `.mcp.json` — they may contain machine-specific paths or secrets.
> The repo `.gitignore` blocks the real filenames as a safety net.

## settings.example.json

Template for your user settings (`~/.claude/settings.json`): default model, TUI mode, and a
`permissions` skeleton to grow allow/deny rules into.

```bash
cp config/settings.example.json ~/.claude/settings.json
# then edit ~/.claude/settings.json to taste
```

## mcp.example.json

Template showing the shape of an MCP server config. Real API keys/tokens should be supplied
via environment variables (`${ENV_VAR}` placeholders), never hard-coded.

```bash
# merge the relevant entries into your real MCP config, then set the env vars it references
```
