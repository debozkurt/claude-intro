# Appendix — Rename / Changelog Log

*Last verified: 2026-04-19 — regenerate from [4] quarterly*

Fast-moving surfaces in Claude Code. Track what's been renamed, defaulted differently, or newly-added since the primer — useful when reading older blog posts or docs.

---

## Renames (2025-2026)

| Old name | New name | Version | When | Notes |
|---|---|---|---|---|
| `/fork` | `/branch` | v2.1.77 | Late 2025 / early 2026 | Same feature — session-duplication. Old docs still use `/fork`. [4] |
| "Claude Code SDK" | "Agent SDK" | — | Late 2025 | Library rebrand. Same harness, broader positioning to cover general-purpose agents. [5] |

---

## Default changes

| What | Old | New | When | Status |
|---|---|---|---|---|
| Prompt cache TTL | 1 hour | 5 minutes (for many CC requests) | April 2026 | **Actively contested** [4]. Flag available in SDK; CLI handling evolving. |

---

## Newly formalized (2025-2026)

| Feature | What changed |
|---|---|
| **`.claude/rules/` path-scoping** | `paths:` YAML frontmatter for file-scoped rules — loads only when Claude touches matching files. Ch 04. |
| **Auto-memory** | Formerly an informal pattern; now a first-class feature at `~/.claude/projects/<slug>/memory/MEMORY.md` with hard 200-line / 25KB budget. Ch 05. |
| **Plan mode** | Formerly a prompt convention; now a first-class permission mode with `--permission-mode plan`. Ch 07, Ch 08. |
| **MCP registry** | Launched 2025 at `registry.modelcontextprotocol.io`. Vendor-neutral discovery. Ch 15. |
| **OpenAI + Google MCP adoption** | March 2025 (OpenAI), mid-2025 (Google DeepMind). MCP is no longer Anthropic-only. Ch 15. |
| **`TRACEPARENT` in headless** | Headless Claude honors the env var for OpenTelemetry context propagation. Ch 23. |

---

## Current feature-flag / evolving territory

These are moving — verify at invocation time.

| Surface | Why watch |
|---|---|
| Cache TTL | Default change (April 2026) still contested; flag for longer TTL may come and go [4]. |
| Skill auto-invoke matching | Matching heuristics are tuned over time; what worked in 2025 may over- or under-fire in 2026 [3]. |
| Agent SDK surface area | Rapid expansion; check current `code.claude.com/docs/en/agent-sdk/` for additions [5]. |
| MCP server availability | Registry is growing and shifting; quality varies. Check maintainer activity before depending [10]. |
| Hook event types | May gain richer payload formats (structured JSON over env vars) in future versions. |

---

## How to stay current

**Monthly:** scan [CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) for the last 4-6 weeks. 5-minute read.

**Quarterly:** re-verify the "Last verified" lines across your own docs. Spot-check that cited behaviors still match current Claude Code. Update as needed.

**When writing new docs:** always stamp with a "Last verified: YYYY-MM-DD" header. Treat any doc without one as suspect until proven otherwise.

---

## Deprecations / removed features

*(None significant as of 2026-04-19.)*

---

## Sources

[3] Equipping agents with Agent Skills — <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
[5] Agent SDK overview — <https://code.claude.com/docs/en/agent-sdk/overview>
[10] Claude Code MCP docs — <https://code.claude.com/docs/en/mcp>
