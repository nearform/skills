# Nearform Skills (Public)

A curated, public subset of [Nearform](https://nearform.com/)'s skills registry — plugins for Claude Code, Cursor, and GitHub Copilot CLI that we've chosen to share with the wider community.

This repository is **automatically generated** from our internal skills registry. Don't open PRs directly here — they'll be overwritten on the next sync. Issues are fine.

## Installing

```
# Claude Code
/plugin marketplace add nearform/skills
/plugin install <plugin>@nearform-skills

# Cursor
Settings → Plugins → Team Marketplaces → add nearform/skills

# GitHub Copilot CLI
copilot plugin marketplace add nearform/skills
```

## What's in a plugin

A plugin is a directory under `plugins/<plugin-name>/` with a `plugin.json` manifest and one or more content items:

| Content type | Path | Description |
|---|---|---|
| **Skill** | `skills/<skill-name>/SKILL.md` | Teaches an agent how to complete a specific task |
| Command | `commands/<command-name>.md` | A slash command the agent can invoke |
| Agent | `agents/<agent-name>.md` | A named agent persona with a defined role |
| Hook | `hooks/<filename>` | A lifecycle hook run at defined trigger points |
| MCP server | `mcp.json` | An MCP server configuration |

## Contributing

Skills are authored in Nearform's private registry. If you'd like to suggest improvements, file an issue here describing what you'd change.

## License

See [LICENSE](./LICENSE).
