# personal-toolkit

Kevin Dayno's personal Claude Code plugin: skills, slash commands, subagents, and hooks.

## Layout

| Folder | What goes here |
|---|---|
| `skills/` | Skills — one folder per skill, each containing a `SKILL.md`. |
| `commands/` | Slash commands — one markdown file per command. |
| `agents/` | Subagent definitions — markdown with frontmatter. |
| `hooks/` | Hook scripts + a `hooks.json` wiring file. |

Claude Code auto-discovers these folders from `.claude-plugin/plugin.json`, so adding a
new skill/command/agent is just dropping a file in the right folder.

## Install

```
/plugin marketplace add kdayno/claude-code-toolkit
/plugin install personal-toolkit@kdayno
```

See the repo root [README](../../README.md) for full details.
