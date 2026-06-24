# Nearform Skills

**Battle-tested skills for AI coding agents, built by [Nearform](https://nearform.com/) engineers, from real client work.**

A skill teaches your coding agent how to do one thing well. We package the hard-won patterns our teams use every day and share a curated set of them here, so your agent can use them too.

---

## Why skills

Coding agents are only as good as what they know about *your* way of working. A general model writes generic code. A skill gives it the specifics: the conventions, the guardrails, the playbook a senior engineer would apply.

- **From real engagements.** Every skill here started as a pattern that solved a real problem on a real project, not a tutorial.
- **Across the whole lifecycle.** Our skills span the full software delivery cycle: design and architecture, scaffolding, writing and reviewing code, testing. They also cover the non-functional concerns that decide whether software survives contact with production: security, performance, reliability, and maintainability.
- **Cross-platform.** The same skill runs on Claude Code, Cursor, and Copilot CLI. No lock-in.
- **Lean by design.** Agents read each skill's summary up front and pull in the full instructions only when a task needs them. A broad library stays cheap on context.

This is a curated, public subset of our internal registry, and it keeps growing as we open up more of what our teams build.

## Installing

**Claude Code**

```bash
/plugin marketplace add nearform/skills
/plugin install <plugin>@nearform-skills
```

**GitHub Copilot CLI**

```bash
copilot plugin marketplace add nearform/skills
copilot plugin install <plugin>
```

**Cursor** (Teams and Enterprise plans)

In the Cursor Dashboard, go to Settings, then Plugins. Under Team Marketplaces, click Import and paste `https://github.com/nearform/skills`.

### Manual install (any agent)

No marketplace support? Skills are just folders of Markdown, so they work with any agent. Clone the repo and copy the skill folder into wherever your agent loads skills from.

```bash
git clone https://github.com/nearform/skills.git
# then copy any skill into your project, for example with Claude Code:
cp -r plugins/<plugin>/skills/<skill> .claude/skills/

Common locations:

- **Claude Code:** `.claude/skills/<skill>/` in a project, or `~/.claude/skills/<skill>/` for every project.
- **Cursor:** add it from the Customize page, or drop the folder into your project's skills directory.
- **Other agents:** put the `SKILL.md` and its supporting files wherever your agent reads skills or rules from. Check your agent's docs for the exact path.

Each skill is self-contained in its own folder, so there is nothing else to wire up.

Browse the available plugins right here in the [`plugins/`](./plugins) directory. Each one ships one or more `SKILL.md` files with its full instructions.

## What's in a plugin

A plugin is the unit you install. Each one lives under `plugins/<plugin-name>/` and pairs a `plugin.json` manifest with one or more content items:

| Content type | Path | Description |
|---|---|---|
| **Skill** | `skills/<skill-name>/SKILL.md` | Teaches an agent how to complete a specific task |
| Command | `commands/<command-name>.md` | A slash command the agent can invoke |
| Agent | `agents/<agent-name>.md` | A named agent persona with a defined role |
| Hook | `hooks/<filename>` | A lifecycle hook run at defined trigger points |
| MCP server | `mcp.json` | An MCP server configuration |

## About Nearform

[Nearform](https://nearform.com/) is an AI-Native engineering consultancy firm. We help organisations build and ship software that matters. These skills are a small, open window into how our teams work.

## Contributing

This repository is **automatically generated** from our internal skills registry. Pull requests opened here will be overwritten on the next sync, but **issues are very welcome**. Found a bug, or have an idea to make a skill better? [Open an issue](https://github.com/nearform/skills/issues).

## License

See [LICENSE](./LICENSE).
