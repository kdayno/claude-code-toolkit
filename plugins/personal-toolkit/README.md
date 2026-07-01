# personal-toolkit

Kevin Dayno's personal Claude Code plugin: skills, slash commands, subagents, and hooks.

## Layout

| Folder | What goes here |
|---|---|
| `skills/` | Skills — one folder per skill, each containing a `SKILL.md`. |
| `commands/` | Slash commands — one markdown file per command (ships `vendor-skill`). |
| `agents/` | Subagent definitions — markdown with frontmatter. |
| `hooks/` | Hook scripts + a `hooks.json` wiring file. |

Claude Code auto-discovers these folders from `.claude-plugin/plugin.json`, so adding a
new skill/command/agent is just dropping a file in the right folder.

## Vendoring external skills

Skills found online (e.g. on GitHub) are **vendored** — copied into `skills/` so this repo stays
self-contained and each copy can be edited freely. To keep track of where a copy came from (and
whether it has changed upstream), each vendored skill gets a `.source.yml` next to its `SKILL.md`
recording the source repo, ref, subpath, exact commit SHA, and date.

Use the `/vendor-skill` command (run it from inside this repo):

```
/vendor-skill add https://github.com/owner/repo/tree/main/skills/some-skill   # import a skill
/vendor-skill list                                                            # what's vendored + pins
/vendor-skill check                                                           # has upstream moved?
/vendor-skill update some-skill                                               # review diff, then adopt
```

Updates are deliberate, never automatic: `check` tells you when upstream has moved, and `update`
shows the diff and asks before overwriting — so an upstream change can't silently alter behavior.
Skills without a `.source.yml` (like `hello-world` and your own authored skills) are left untouched.

## Install

```
/plugin marketplace add kdayno/claude-code-toolkit
/plugin install personal-toolkit@kdayno
```

See the repo root [README](../../README.md) for full details.
