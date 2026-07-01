---
description: Vendor an external Claude Code skill into this toolkit with source provenance.
argument-hint: "add <github-url> [name] | check | update <name> | list"
allowed-tools: Bash(git:*), Bash(mkdir:*), Bash(cp:*), Bash(rm:*), Read, Write, Glob
---

You are running the `/vendor-skill` command, which imports ("vendors") an external Claude Code
skill into this repo and tracks where it came from so it can be updated deliberately later.

Arguments: `$ARGUMENTS`

The first word is the subcommand (`add`, `check`, `update`, or `list`). If it is missing or
unrecognized, print the usage line from `argument-hint` above and stop.

## Step 0 — resolve the target directory (always do this first)

Run `git rev-parse --show-toplevel` to find the repo root. Require that this is the
`claude-code-toolkit` repo — it must contain `plugins/personal-toolkit/`. If it does not, STOP and
tell the user to `cd` into the toolkit repo first (vendored skills are authored in the repo working
tree, not in the installed plugin cache; do **not** use `${CLAUDE_PLUGIN_ROOT}` — that points at the
read-only installed copy).

Let `SKILLS_DIR = <repo-root>/plugins/personal-toolkit/skills`.

## The provenance file

Every vendored skill has a `.source.yml` beside its `SKILL.md`. Format (flat `key: value`, no
nesting so it stays grep-parseable):

```yaml
source_repo:   https://github.com/owner/repo
source_ref:    main
source_path:   skills/<name>        # folder within the source repo, or "." for the repo root
source_commit: <full 40-char sha>
vendored_on:   <YYYY-MM-DD>
notes:         ""                    # optional: local edits made after vendoring
```

A skill folder **without** a `.source.yml` is locally authored (e.g. `hello-world`) — skip it in
`check`, `update`, and `list`.

## Subcommand: `add <github-url> [name]`

1. **Parse the URL** into `owner/repo`, `ref`, and `subpath`:
   - `https://github.com/OWNER/REPO/tree/REF/SUB/PATH` → repo `OWNER/REPO`, ref `REF`, subpath `SUB/PATH`.
   - A bare `https://github.com/OWNER/REPO` (optionally `.git`) → ref = default branch (use `main`;
     if the clone fails, retry with `master`), subpath = `.`.
   - The clone URL is `https://github.com/OWNER/REPO.git`.
2. **Sparse shallow clone** into the session scratchpad so a large monorepo only downloads the one
   folder. Use a temp dir under the scratchpad path, e.g. `TMP=<scratchpad>/vendor-<name>`:
   ```
   git clone --depth 1 --branch <ref> --filter=blob:none --sparse <clone-url> "$TMP"
   git -C "$TMP" sparse-checkout set <subpath>
   ```
   (If subpath is `.`, skip the `sparse-checkout set` line.)
3. **Capture the commit:** `git -C "$TMP" rev-parse HEAD`.
4. **Choose the name:** use the `[name]` arg if given, else the basename of `<subpath>` (or the repo
   name if subpath is `.`). If `SKILLS_DIR/<name>/` already exists, show the user what's there and
   ask before overwriting.
5. **Copy in and record provenance:**
   - `mkdir -p "SKILLS_DIR/<name>"` and copy the source folder's contents into it
     (`cp -R "$TMP/<subpath>/." "SKILLS_DIR/<name>/"`).
   - Write `SKILLS_DIR/<name>/.source.yml` with the fields above (use today's date for `vendored_on`).
   - Read the vendored `SKILL.md`, then summarize for the user what the skill does. Glob the other
     skills' `SKILL.md` frontmatter and warn about any `name` clash or overlapping
     `description`/trigger wording.
6. **Clean up:** `rm -rf "$TMP"`. Remind the user to review the files, then `git add` + commit, and
   run `/plugin marketplace update kdayno` to pick it up in installed sessions.

## Subcommand: `check`

Glob `SKILLS_DIR/*/.source.yml`. For each, read `source_repo`, `source_ref`, and `source_commit`,
then get the current upstream SHA **without cloning**:
`git ls-remote <source_repo> <source_ref>` (take the first column).

Print a table of skill → pinned SHA (short) → upstream SHA (short) → `up to date` or
`⚠ update available`. After the table, note this limitation: the comparison is repo-level, so a
skill vendored from a monorepo can show "update available" even when a *different* skill in that
repo changed — `update` will show the real diff for the skill's own path.

## Subcommand: `update <name>`

1. Read `SKILLS_DIR/<name>/.source.yml`. If missing, tell the user it's a locally authored skill and stop.
2. Re-fetch upstream at `source_ref` using the same sparse-clone steps as `add` (into a temp dir),
   and get the new HEAD SHA.
3. Show the user the diff for this skill's path between the pinned `source_commit` and the new SHA:
   `git -C "$TMP" diff <source_commit> HEAD -- <source_path>` (fall back to a file-level `diff`
   against the current vendored copy if the pinned commit isn't reachable in the shallow clone).
4. **Ask for confirmation.** On decline, clean up and change nothing.
5. On confirm, overwrite the files from the temp clone, update `source_commit` and `vendored_on` in
   `.source.yml` (preserve any `notes`), clean up the temp dir, and remind the user to commit.

## Subcommand: `list`

Glob `SKILLS_DIR/*/.source.yml` and print a table: skill name, `source_repo`, and pinned
`source_commit` (short) + `vendored_on`. If none exist, say so.
