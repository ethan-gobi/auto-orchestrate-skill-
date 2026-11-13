# auto-orchestrator

A [Claude Code](https://claude.com/claude-code) skill that plans and runs complex, multi-step tasks by chaining the skills, plugins, MCP connectors and CLI tools you already have installed — while keeping token use low.

## What it does

- **Takes inventory** of the skills, MCP tools and CLIs actually available, and falls back gracefully when one is missing.
- **Plans** multi-step work as a visible task list, routing each step to the best existing tool.
- **Delegates down the model ladder** (Fable → Opus → Sonnet → Haiku). The model you're running on keeps planning and judgment, and hands each chunk to the cheapest subagent model below it that can do the job: Haiku for search, reading and mechanical edits, Sonnet for normal implementation and tests, Opus (when you're on Fable) for hard sub-problems. Failed chunks move one rung up.
- **Plan-file handoff for huge jobs** — writes a self-contained `PLAN.md` you can run in a fresh, cheaper session.
- **Keeps context lean** — maps codebases with a structural index (e.g. Graphify) before reading files, delegates bulk reading to subagents, writes large data to files, and suggests `/compact` when context gets heavy.
- **Asks before anything destructive or outward-facing** (deleting files, `git push`, sending messages, installing skills) and moves on its own for read-only steps.

Works best alongside (all optional): the token-saver plugin, Graphify MCP, Composio MCP, and the find-skills skill.

## Install

Personal (all projects):

```bash
git clone https://github.com/ethan-gobi/auto-orchestrate-skill-.git ~/.claude/skills/auto-orchestrator
```

Single project:

```bash
git clone https://github.com/ethan-gobi/auto-orchestrate-skill-.git .claude/skills/auto-orchestrator
```

Restart Claude Code (or start a new session) so it picks up the skill.

**Claude chat (claude.ai / desktop app):** download this repo as a ZIP (Code → Download ZIP) and add it as a skill in Claude's settings. The frontmatter only uses `name` and `description`, so the same file works in both places.

## Use

Claude will use it automatically for multi-step, multi-tool requests. You can also call it directly, in Claude Code or chat:

```
/auto-orchestrator migrate the settings page to the new API and update the tests
```

## Update

```bash
cd ~/.claude/skills/auto-orchestrator && git pull
```
