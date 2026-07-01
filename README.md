# claude-code-toolkit

Kevin Dayno's personal [Claude Code](https://claude.com/claude-code) customizations —
skills, slash commands, subagents, and hooks — kept in one versioned place and installable
on any machine.

This repo is a **self-hosted plugin marketplace**: it is both a catalog (`kdayno`) and the
plugin it ships (`personal-toolkit`). "Marketplace" here just means a catalog file in this
repo — nothing is published to any external registry.

## Install

```text
/plugin marketplace add kdayno/claude-code-toolkit
/plugin install personal-toolkit@kdayno
```

To update after pushing changes:

```text
/plugin marketplace update kdayno
```

Browse and enable/disable installed plugins anytime with `/plugin`.

## What's inside

```
.claude-plugin/marketplace.json     # the catalog (lists the plugin)
plugins/personal-toolkit/           # the plugin
  ├── skills/                       # skills (each a folder with SKILL.md)
  ├── commands/                     # slash commands
  ├── agents/                       # subagents
  └── hooks/                        # hooks
config/                             # sanitized templates for settings.json & MCP config
```

The `personal-toolkit` plugin currently ships a `hello-world` example skill — replace it
with your own. To import skills found online, use the `/vendor-skill` command, which copies a
skill in and records its source for deliberate updates — see the
[plugin README](./plugins/personal-toolkit/README.md#vendoring-external-skills).

## Config templates

`settings.json` and MCP config are not part of a plugin, so sanitized templates live in
[`config/`](./config). **This repo is public — never commit real secrets.** See
[`config/README.md`](./config/README.md).

## License

[MIT](./LICENSE)
