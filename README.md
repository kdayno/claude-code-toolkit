# claude-code-toolkit

Kevin Dayno's personal [Claude Code](https://claude.com/claude-code) customizations —
skills, slash commands, subagents, and hooks — kept in one versioned place and installable
on any machine.

This repo is a **self-hosted plugin marketplace**: it is both a catalog (`kdayno`) and the
plugins it ships (`toolkit-core`, `git-tools`, `creative`, `writing`). "Marketplace" here just means a
catalog file in this repo — nothing is published to any external registry.

## Install

```text
/plugin marketplace add kdayno/claude-code-toolkit
/plugin install toolkit-core@kdayno      # then git-tools@kdayno, creative@kdayno as desired
```

To update after pushing changes:

```text
/plugin marketplace update kdayno
```

Browse and enable/disable installed plugins anytime with `/plugin`.

## What's inside

```
.claude-plugin/marketplace.json     # the catalog (lists all plugins below)
plugins/
  ├── toolkit-core/                 # /vendor-skill command + hello-world example
  ├── git-tools/                    # git workflow skills (git-commit, secret-scanning)
  ├── creative/                     # generative-art skills (algorithmic-art)
  └── writing/                      # writing / prose skills
config/                             # sanitized templates for settings.json & MCP config
```

The toolkit is split into several small plugins so you can **install only what you want** on a given
machine — e.g. `/plugin install git-tools@kdayno` without pulling in the rest:

```
/plugin install toolkit-core@kdayno
/plugin install git-tools@kdayno
/plugin install creative@kdayno
```

To import skills found online, use the `/vendor-skill` command (in `toolkit-core`), which copies a
skill into a chosen plugin and records its source for deliberate updates — see the
[toolkit-core README](./plugins/toolkit-core/README.md#vendoring-external-skills).

## Config templates

`settings.json` and MCP config are not part of a plugin, so sanitized templates live in
[`config/`](./config). **This repo is public — never commit real secrets.** See
[`config/README.md`](./config/README.md).

## License

[MIT](./LICENSE)
