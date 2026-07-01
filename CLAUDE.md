# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **self-hosted Claude Code plugin marketplace**. It is simultaneously a catalog (`kdayno`) and the
plugins it ships. "Marketplace" just means the catalog file `.claude-plugin/marketplace.json` — nothing
is published to any external registry. Users consume it with:

```
/plugin marketplace add kdayno/claude-code-toolkit
/plugin install <plugin>@kdayno          # toolkit-core | git-tools | creative
```

There is **no build, test, or lint step** — content is Markdown (`SKILL.md`, commands, agents) and JSON
(manifests). "Development" means editing those files and validating JSON. The only executable logic is
inside slash-command / skill prompts, which run as instructions to Claude at invocation time (not as
code in this repo).

## Architecture

A plugin is any folder under `plugins/` containing `.claude-plugin/plugin.json`. Claude Code
auto-discovers that plugin's `skills/`, `commands/`, `agents/`, and `hooks/` subfolders — so adding a
capability is just dropping a file in the right place; no wiring or registration.

The repo is deliberately split into **multiple small plugins** (not one big one) because a plugin is the
atomic unit of install/enable in Claude Code — you cannot install or toggle an individual skill within a
plugin. Splitting lets users install only what they want per machine.

```
.claude-plugin/marketplace.json   # lists all plugins (name + relative source path)
plugins/
  toolkit-core/   # the /vendor-skill command + hello-world example
  git-tools/      # git-commit, secret-scanning skills
  creative/       # algorithmic-art skill
  writing/        # writing / prose skills
config/           # sanitized *.example.json templates for user settings.json + MCP config
```

When adding a new plugin, create `plugins/<name>/.claude-plugin/plugin.json` **and** add a corresponding
entry to `marketplace.json` — both are required.

## Vendoring: how external skills get in here (the core concept)

Skills sourced from other repos are **vendored** — copied into a plugin's `skills/<name>/` folder so this
repo is self-contained and each copy is freely editable. Provenance is tracked in a `.source.yml` file
beside each vendored skill's `SKILL.md`:

```yaml
source_repo:   https://github.com/owner/repo
source_ref:    main
source_path:   skills/<name>     # folder within the source repo, or "." for repo root
source_commit: <full sha>        # the exact commit vendored
vendored_on:   <YYYY-MM-DD>
notes:         ""
```

- A skill folder **without** `.source.yml` is locally authored (e.g. `hello-world`) — leave it out of
  update/check tooling.
- The `/vendor-skill` command (in `toolkit-core/commands/vendor-skill.md`) automates add/list/check/update.
  It sparse-shallow-clones only the target subfolder, writes `.source.yml`, and targets a plugin group via
  `--into <group>`. Its behavior is defined entirely by that prompt file — edit the file to change it.
- Updates are **deliberate, never automatic**: `check` compares the pinned commit vs upstream (repo-level,
  so monorepo skills can false-positive); `update` shows a diff and asks before overwriting.
- Aggregator URLs (e.g. skills.sh) are not directly cloneable — resolve them to the real
  `github.com/owner/repo` + path first (use `gh api repos/<owner>/<repo>/git/trees/<ref>?recursive=1` to
  find the skill's subpath).

## Contribution workflow (required)

`main` is **branch-protected**: no direct pushes (enforced even for admins), all changes go through a PR
(0 approvals required, so you can self-merge). The loop:

```
git switch -c feat/<something>
# edit files
git commit ...                              # Conventional Commits (see the git-commit skill)
git push -u origin feat/<something>
gh pr create ...
gh pr merge <n> --merge --delete-branch     # then: git switch main && git pull --ff-only
```

After merging, run `/plugin marketplace update kdayno` (and reinstall the affected plugin) to pick up
changes in an installed session.

- **Commit author identity:** commit as `Kevin Dayno <kdayno@users.noreply.github.com>` (the git config is
  already set to this; do not use the session's other email).
- **Validate JSON before committing:** `jq -e . .claude-plugin/marketplace.json` and
  `jq -e . plugins/*/.claude-plugin/plugin.json`.

## Config templates

`config/` holds only sanitized `*.example.json` templates for config that lives *outside* plugins (your
user `~/.claude/settings.json` and MCP config). **This repo is public.** Never commit real
`settings.json` / `.mcp.json` or secrets — `.gitignore` blocks the real filenames as a safety net, and MCP
configs must reference secrets via `${ENV_VAR}` placeholders, never hard-coded values.
