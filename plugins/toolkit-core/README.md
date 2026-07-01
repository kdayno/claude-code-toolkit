# toolkit-core

Core / meta tooling for Kevin Dayno's Claude Code toolkit — the `/vendor-skill` command plus a
`hello-world` example skill.

## Install

```
/plugin install toolkit-core@kdayno
```

## Contents

| Folder | What |
|---|---|
| `commands/` | `vendor-skill` — import external skills into any plugin in this repo. |
| `skills/` | `hello-world` — minimal example to verify the marketplace installs and runs. |

## Vendoring external skills

Skills found online (e.g. on GitHub) are **vendored** — copied into a plugin's `skills/` folder so
this repo stays self-contained and each copy can be edited freely. To keep track of where a copy came
from (and whether it has changed upstream), each vendored skill gets a `.source.yml` next to its
`SKILL.md` recording the source repo, ref, subpath, exact commit SHA, and date.

Use the `/vendor-skill` command (run it from inside this repo). Because the repo now has multiple
plugins, `add` takes a target group via `--into`:

```
/vendor-skill add https://github.com/owner/repo/tree/main/skills/some-skill --into git-tools
/vendor-skill list                        # what's vendored across all groups, + pins
/vendor-skill check                       # has upstream moved?
/vendor-skill update some-skill           # review diff, then adopt
```

If you omit `--into`, the command lists the available groups and asks which one. Updates are
deliberate, never automatic: `check` tells you when upstream has moved, and `update` shows the diff
and asks before overwriting — so an upstream change can't silently alter behavior. Skills without a
`.source.yml` (like `hello-world` and your own authored skills) are left untouched.
