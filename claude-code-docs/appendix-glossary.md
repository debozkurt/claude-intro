# Appendix — Glossary

*Last verified: 2026-04-19*

Terms that show up repeatedly across this guide. Brief definitions — chapters linked have the depth.

---

**Agent harness.** The runtime around an LLM that handles the loop, context assembly, tool dispatch, and state. Claude Code is one. [Ch 01]

**Agent SDK.** The Python / TypeScript library that exposes Claude Code's harness programmatically. Same harness as the CLI; different I/O. Formerly called the "Claude Code SDK." [Ch 18]

**Auto memory.** Claude-authored notes that persist across sessions, stored as markdown under `~/.claude/projects/<slug>/memory/`. First 200 lines / 25KB of `MEMORY.md` loaded every session. [Ch 05]

**Bypass (permission mode).** `--dangerously-skip-permissions`. Skips every permission prompt. Sandboxed environments only. Not in the `Shift+Tab` cycle. [Ch 07]

**Cache TTL.** Time-to-live of the prompt cache. Dropped to 5 min default in April 2026 [4]. Affects how long idle periods between turns stay cheap. [Ch 06, Ch 22]

**CLAUDE.md.** Markdown file(s) Claude loads as persistent context at every session start. Hierarchy: user (`~/.claude/CLAUDE.md`), project (`<repo>/CLAUDE.md`), subfolder (lazy-loaded when Claude touches the dir). [Ch 03]

**Fork.** Old name for `/branch`. Renamed v2.1.77. Conversation forking = duplicating a session JSONL to evolve both paths. [Ch 09]

**Headless mode.** Non-interactive CLI invocation via `claude -p "task"`. Single prompt, streams to stdout, exits. Foundation for automation. [Ch 17]

**Hook.** Shell command registered in `settings.json` that fires on lifecycle events (`PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `SessionStart`, `Notification`, `PreCompact`). Deterministic, no model involvement. [Ch 14]

**MCP (Model Context Protocol).** Open protocol for exposing tools, resources, and prompts to LLM clients. Portable across Claude Code, Cursor, OpenAI, etc. [Ch 15, Ch 16]

**`.mcp.json`.** Project-scoped MCP server config at `<repo>/.claude/.mcp.json`. Committed; team-shared. [Ch 16]

**Plan mode.** Read-only permission mode. Claude investigates and proposes a plan but doesn't edit or run state-changing commands. [Ch 08]

**Progressive disclosure.** Skill loading model — frontmatter always in context, body only loaded when the skill is invoked. Enables many installed skills without blowing context. [Ch 12]

**Prompt cache.** Anthropic's caching of stable prefix content. Cache reads ~10% the cost of cache writes. [Ch 06, Ch 22]

**Ralph loop.** Long-running agent loop pattern: queue → iterate → externalize state → repeat. Named by the agent-theory lessons in `agents/29`. [Ch 20]

**Rule file.** Markdown file under `.claude/rules/` — can be path-scoped via YAML `paths:` frontmatter. Loads alongside CLAUDE.md with same priority. [Ch 04]

**Session JSONL.** Plaintext JSONL file per conversation at `~/.claude/projects/<slug>/<uuid>.jsonl`. One JSON object per turn. Powers `/resume`, `/rewind`, `/branch`. [Ch 02, Ch 09]

**Skill.** Markdown file with YAML frontmatter under `.claude/skills/<name>/SKILL.md`. Invoked by user (`/<name>`) or auto-matched by Claude from description. Can declare its own tool budget. [Ch 12]

**Slash command.** Prompt template at `.claude/commands/<name>.md`. User-invoked only via `/<name>`. Simpler than a skill — no model matching, no tool restrictions. [Ch 11]

**Subagent.** Delegate Claude invoked via the parent's `Agent` tool. Fresh context window, restricted tools, only final output returns to parent. Definitions live at `.claude/agents/<name>.md`. [Ch 13, Ch 19]

**`ultrathink`.** Practitioner magic string that signals maximum extended-thinking budget. Works because the harness recognizes it, not because the model does. Official SDK equivalent is the `thinking_budget` parameter. [Ch 21]

**Worktree.** Git concept — separate working directory attached to the same `.git` object DB with its own branch. Claude Code keys session state per-directory, so worktrees automatically yield isolated sessions. [Ch 10]

**YOLO.** Community slang for bypass-permission mode. Officially deprecated framing; prefer "bypass permissions." [Ch 07, Ch 24]
