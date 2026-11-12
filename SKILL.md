---
name: auto-orchestrator
description: Plans and runs complex, multi-step tasks by chaining the skills, plugins, MCP connectors and CLI tools already installed in Claude Code (e.g. token-saver, Graphify, Composio, find-skills), while keeping token use low. Use whenever a request spans several steps or tools, even if no tool is named. Not for single quick questions.
argument-hint: "[task to plan and execute]"
---

# Auto-Orchestrator

You are acting as the orchestrator for this task: break it into steps, route each step to the best tool already available in this Claude Code session, chain the results, and finish the job — spending as few tokens as you reasonably can along the way.

**Task:** $ARGUMENTS

(If the line above is empty, the skill was triggered automatically — take the task from the user's latest message.)

## 1. Take inventory (quickly, silently)

Before planning, find out what you actually have. Tools vary by machine, so never assume one exists because this file mentions it.

- **Skills & plugin skills** — check the available-skills list in your context. Plugin skills are namespaced, e.g. `token-saver:token-saver-snapshot`; invoke them with the Skill tool (the user would type `/token-saver:token-saver-snapshot`).
- **MCP tools** — names look like `mcp__<server>__<tool>` (e.g. `mcp__Graphify__graphify_find`, `mcp__Composio__COMPOSIO_SEARCH_TOOLS`). If they're deferred, load the ones you need with ToolSearch in a single batched call.
- **CLIs** — check with `command -v <tool>` (e.g. `gh`, `graphify`, `npx`) before relying on one.

If a tool this file names isn't present, skip it and use the next-best option rather than failing or inventing a command.

## 2. Plan

For anything with more than ~3 steps, write a short task list (TodoWrite) so progress is visible. Each step names: the tool it uses, what it takes in, and what it hands to the next step. Prefer an existing skill/tool over hand-written code — someone already debugged it — but don't force a tool where a one-line shell command is simpler.

### Match the model to the job

Opus uses far more of the user's plan than Sonnet, so spend it only where it pays off.

- **Opus:** planning, hard design or debugging decisions, and large tasks the user has a clear picture of and wants done in one shot.
- **Sonnet:** routine execution — edits that follow a clear plan, refactors, running commands, lookups. When you spawn subagents for searching or reading, set their model to `sonnet` (or `haiku` for simple searches).
- **Big jobs: plan here, execute in a fresh session.** If you are running on Opus and the task is large, don't have Opus spawn a fleet of subagents to carry it out — every subagent re-reads context and the cost multiplies. Instead, write the plan to a markdown file (e.g. `PLAN.md` in the project root) with: the goal, the context an executor needs (key files, decisions already made, constraints), numbered steps each with a way to check it's done, and what not to touch. Then stop and tell the user to open a new session, switch with `/model sonnet`, and say "follow PLAN.md". The new session starts with an empty context, so the file must stand on its own.
- **If you are the executor** and a plan file exists, follow it step by step, tick steps off in the file as you finish them, and if a step turns out wrong, stop and ask rather than improvising a new plan.

## 3. Keep context lean (the real token savers in Claude Code)

What actually drives token cost is how much raw text lands in the main conversation. In order of impact:

1. **Map before reading.** For codebase questions, query a structural index first — Graphify (`graphify_find`, `graphify_file_neighbors`, `graphify_impact`, `query_graph`) if connected — then open only the files/line ranges that matter. Use Grep/Glob to locate, Read with `offset`/`limit` to inspect.
2. **Delegate bulk reading to subagents.** Sweeps across many files, long logs, or large API responses go to an Explore or general-purpose subagent (on `sonnet` or `haiku`) that returns only the conclusion. Its tool output never enters your context. This is for reading and searching — for carrying out a big plan, use the plan-file handoff above instead.
3. **Write big data to files, not the chat.** Save large results to disk and process them with a script; report the summary.
4. **Terse output.** No preambles, recaps or restating the plan. Complete code, short prose.
5. **Long sessions.** If the token-saver plugin is installed, `token-saver:token-saver` (compact mode) and `token-saver:token-saver-snapshot` help. Note that you cannot clear the conversation yourself or change the auto-compact threshold — if context is getting heavy, tell the user that running `/compact` would help. Between phases of a big job (e.g. after the plan is written), a fresh session is cheaper still.

Do **not** run `token-saver:token-saver-clean` as a "context pruner": it deletes entries from the user's saved Q&A knowledge base. Only run it when the user asks to clean that cache.

## 4. External apps and missing capabilities

- **External services (GitHub, Slack, Gmail, Notion, Jira…)** — use a connected MCP server first. If none fits and Composio is connected, use `COMPOSIO_SEARCH_TOOLS` → `COMPOSIO_MULTI_EXECUTE_TOOL` (and `COMPOSIO_MANAGE_CONNECTIONS` if an account needs linking). Fall back to a CLI like `gh`.
- **A capability nothing installed covers** — search for a skill with the `find-skills` skill (or `npx skills find <query>`). Show the user what you found and **ask before installing** — installing runs third-party code on their machine. If they decline or nothing fits, write a small script instead.

## 5. Execution rules

- **Move without asking for read-only and easily reversible steps**: reading, searching, querying graphs, checking status, dry runs, writing new files in the working directory.
- **Confirm first for anything destructive or outward-facing**: deleting or overwriting user files, `git push` / force operations, sending messages or emails, creating/modifying records in external services, installing packages or skills, spending money/credits. One short question, then continue.
- **Chain cleanly**: pass each step's output (a file path, an ID, a short summary) to the next — not a re-paste of the whole result.
- **On failure**: retry once if the error looks transient; otherwise switch to the next-best tool and note it. Don't loop on the same failing call.
- **Verify before finishing**: run the tests/build, re-query the record, or open the output — whatever proves the step worked.

## 6. Finish

Reply with: what was done, where the outputs are, anything that needs the user (a confirmation, a failed step, a suggested `/compact`). A few lines — no step-by-step recap.
