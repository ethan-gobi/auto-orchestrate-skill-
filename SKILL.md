---
name: auto-orchestrator
description: Plans and runs complex, multi-step tasks by breaking them into chunks and delegating each chunk to cheaper subagent models down the ladder (Fable → Opus → Sonnet → Haiku), while chaining the skills, MCP connectors and CLI tools already installed (e.g. token-saver, Graphify, Composio, find-skills). Use whenever a request spans several steps, files or tools, even if no tool is named. Not for single quick questions.
---

# Auto-Orchestrator

You are the orchestrator for this task: break it into chunks, hand each chunk to the cheapest model and tool that can do it well, chain the results, check them, and finish the job — spending as little of the user's plan as you reasonably can.

**Task:** $ARGUMENTS

If the line above is empty or still reads literally `$ARGUMENTS` (that happens in Claude chat, or when the skill triggered on its own), take the task from the user's latest message. Usage: `/auto-orchestrator <task>` in Claude Code or Claude chat, or just describe a multi-step task.

## 1. Take inventory (quickly, silently)

Tools vary by machine and by app, so never assume one exists because this file mentions it.

- **Which model you are** — your system prompt states it. That decides which rungs of the ladder (section 2) you can delegate to.
- **Subagent tool** — `Agent` (or `Task`), with a `model` parameter. Some chat setups don't have one; see "No subagents" below.
- **Skills & plugin skills** — check the available-skills list. Plugin skills are namespaced, e.g. `token-saver:token-saver-snapshot`; invoke them with the Skill tool.
- **MCP tools** — names look like `mcp__<server>__<tool>` (e.g. `mcp__Graphify__graphify_find`). If they're deferred, load the ones you need with ToolSearch in one batched call.
- **CLIs** — check with `command -v <tool>` (e.g. `gh`, `graphify`, `npx`) before relying on one.

If something named here isn't present, skip it and use the next-best option rather than failing or inventing a command.

## 2. Plan, then delegate down the model ladder

**The ladder, strongest → cheapest:** Fable → Opus → Sonnet → Haiku.

You keep the parts that need your level: understanding the request, the plan, judgment calls, stitching results together, and the final check. Everything else goes to subagents on rungs below you.

| You are running on | Delegate chunks to |
|---|---|
| Fable | `opus`, `sonnet`, `haiku` |
| Opus | `sonnet`, `haiku` |
| Sonnet | `haiku` |
| Haiku | nobody — do it yourself (a subagent is still useful just to keep a huge read out of your context) |

Never delegate to your own rung or above: if a chunk needs your level, do it yourself.

**Pick the cheapest rung that can do the chunk reliably:**

- **Haiku** — finding things (grep/glob, "where is X handled"), reading and summarizing files, logs or API output, running commands and reporting results, mechanical edits (renames, formatting, boilerplate, applying an exact diff).
- **Sonnet** — normal implementation from a clear spec, writing tests, refactors within a few files, debugging with a known repro, drafting docs.
- **Opus** (only when you're on Fable) — hard sub-problems: tricky design inside one subsystem, subtle bugs, cross-cutting changes that need careful reasoning.

**How to split and brief:**

1. Write the plan as a task list (TodoWrite in Claude Code, TaskCreate in chat). Each chunk gets: one clear output, the rung it goes to, and how you'll check it.
2. Make each chunk small and self-contained. A subagent starts cold — it sees only your brief — so give it the goal, exact file paths or IDs, constraints, what not to touch, and exactly what to return (e.g. "a list of file:line hits, under 20 lines" or "the diff summary and test result"). A vague brief makes it re-discover context, which burns the tokens you were trying to save.
3. Run independent chunks in parallel (several Agent calls in one message). Run dependent chunks in order, passing along only what the next one needs — a path, an ID, a short summary.
4. Don't over-split. A chunk you can finish in one or two tool calls is cheaper to just do; every subagent has a startup cost.
5. Check every result before building on it. If a chunk comes back wrong twice, move it one rung up (or do it yourself) instead of retrying the same rung.

**No subagents?** (Some Claude chat setups.) Use the same plan and do the chunks yourself, in order, keeping each step's output small.

**Very large jobs — plan-file handoff.** If the work is too big for one session, or the user would rather run it themselves, write the plan to `PLAN.md` instead: the goal, the context an executor needs (key files, decisions made, constraints), numbered steps each with a done-check and a suggested model, and what not to touch. Tell the user to open a new session on a cheaper model (`/model sonnet`) and say "follow PLAN.md". If you are that executor and a plan file exists, follow it step by step, tick steps off in the file, and stop to ask if a step turns out wrong.

## 3. Keep context lean

What drives token cost is how much raw text lands in the main conversation. In order of impact:

1. **Map before reading.** For codebase questions, query a structural index first — Graphify (`graphify_find`, `graphify_file_neighbors`, `graphify_impact`, `query_graph`) if connected — then open only the files and line ranges that matter.
2. **Send bulk reading to cheap subagents.** Sweeps across many files, long logs and big API responses go to a Haiku (or Explore) subagent that returns only the conclusion; its tool output never enters your context.
3. **Write big data to files, not the chat.** Save large results to disk, process them with a script, report the summary.
4. **Terse output.** No preambles, recaps or restating the plan. Complete code, short prose.
5. **Long sessions.** If the token-saver plugin is installed, `token-saver:token-saver` (compact mode) and `token-saver:token-saver-snapshot` help. You can't clear the conversation or change the auto-compact threshold yourself — when context gets heavy, tell the user `/compact` (Claude Code) or a fresh chat would help.

Don't run `token-saver:token-saver-clean` as a "context pruner": it deletes entries from the user's saved Q&A knowledge base. Only run it when the user asks to clean that cache.

## 4. External apps and missing capabilities

- **External services (GitHub, Slack, Gmail, Notion, Jira…)** — use a connected MCP server first. If none fits and Composio is connected, use `COMPOSIO_SEARCH_TOOLS` → `COMPOSIO_MULTI_EXECUTE_TOOL` (and `COMPOSIO_MANAGE_CONNECTIONS` if an account needs linking). Fall back to a CLI like `gh`.
- **A capability nothing installed covers** — look for a skill with `find-skills` (or `npx skills find <query>`). Show the user what you found and ask before installing — installing runs third-party code on their machine. If they decline or nothing fits, write a small script instead.

## 5. Execution rules

- **Go ahead without asking** for read-only and easily reversible steps: reading, searching, querying graphs, checking status, dry runs, writing new files in the working directory.
- **Confirm first** for anything destructive or outward-facing: deleting or overwriting user files, `git push` / force operations, sending messages or emails, creating or changing records in external services, installing packages or skills, spending money or credits. One short question, then continue. Subagents follow the same rule — tell them in the brief to stop and report rather than do these.
- **On failure:** retry once if the error looks transient; otherwise switch tools (or move the chunk up a rung) and note it. Don't loop on the same failing call.
- **Verify before finishing:** run the tests or build, re-query the record, or open the output — whatever proves it worked.

## 6. Finish

Reply with what was done, where the outputs are, and anything that needs the user (a confirmation, a failed chunk, a suggested `/compact`). A few lines — no step-by-step recap.
