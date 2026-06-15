# agentbrowse — Claude Code plugin & skill

Make your coding agent **drive any website from the terminal** by default.

This repo packages the [`agentbrowse`](https://www.npmjs.com/package/agentbrowse) skill as a Claude Code **plugin** and a single-plugin **marketplace**. Installing it teaches your agent to reach for `npx agentbrowse` whenever a task needs a live, interactive page — opening URLs, clicking, filling forms, reading JavaScript-rendered content, and reusing a login.

## Install in Claude Code

```text
/plugin marketplace add mandarwagh9/agentbrowse-skill
/plugin install agentbrowse@agentbrowse
```

That's it — Claude now invokes the skill automatically on web-interaction tasks.

## Other agents (Codex, Cursor, Gemini, Windsurf)

The `agentbrowse` CLI installs the skill into each agent's native format for you:

```bash
npx agentbrowse skill           # auto-detect the agents configured in this project
npx agentbrowse skill --global  # or install once at the user level
npx agentbrowse skill cursor    # or target a specific agent
```

It writes a `SKILL.md` for Claude Code, a `.cursor/rules` rule for Cursor, and an `AGENTS.md` block for Codex and others. It's idempotent and preserves existing content. Preview with `--print`.

## What is agentbrowse?

A CLI that drives a real browser (Playwright) from the shell, exposing a clean, parseable surface — `open`, `snapshot`, `click`, `read`, `fill`, `login`, … — backed by a persistent browser session that survives across commands. See the bundled [`SKILL.md`](skills/agentbrowse/SKILL.md) and [`reference.md`](skills/agentbrowse/reference.md), and the package on [npm](https://www.npmjs.com/package/agentbrowse).

## Licensing

This repo (the agent-facing skill, plugin manifest, and marketplace) is **MIT-licensed** so anyone can add the marketplace freely. The agentbrowse engine itself is distributed via npm (`npx agentbrowse`) under its own license.
