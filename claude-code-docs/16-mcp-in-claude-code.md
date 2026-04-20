# Chapter 16 — MCP Inside Claude Code

*Last verified: 2026-04-19 — Prerequisites: Ch 15 — Status: MCP*

**Builds on:** [`15-mcp-protocol.md`](15-mcp-protocol.md). **Fresh ground** for Claude-Code-specific scoping, auth, and trust model.

---

## Concept

Claude Code loads MCP server configurations at **three scopes** — user, project, local — each serving a different purpose [10]. Servers start as subprocesses (or connect to HTTP endpoints), expose their tools to the active session, and appear alongside Claude Code's native tools in the model's available toolset.

This chapter covers the Claude-Code-specific mechanics: where configs live, how auth flows, what `/mcp` does, what happens on failure, and when to prefer an MCP server over a skill.

## How it works

### Three config scopes

| Scope | Location | Purpose | Committed? |
|---|---|---|---|
| **User** | `~/.claude/settings.json` → `mcpServers` | Your personal servers, every project | No |
| **Project** | `<repo>/.claude/.mcp.json` | Team-shared servers for this repo | Yes |
| **Local** | `<repo>/.claude/settings.local.json` → `mcpServers` | Your personal project-specific | No |

All three merge at session start. Same server name at higher-specificity scope overrides lower.

### A typical project `.mcp.json`

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "${DATABASE_URL}"]
    }
  }
}
```

Env variables expand from your shell — secrets live in your shell env, not in committed config.

### The `/mcp` command

Inside a session:

```
/mcp
```

Shows connected servers, their status, and the tools they expose. Use this first when an MCP tool isn't working — often the server isn't connected, or auth failed, or the tool namespace is different from what you expected.

### Auth flows

Three common patterns [10]:

1. **Env variable** — token in your shell env, substituted at launch (shown above)
2. **OAuth handshake** — server initiates OAuth on first tool call; session completes it, caches the token
3. **Managed credentials** — enterprise/cloud deployments with a secret manager; configure via MCP server's own docs

Security implication: an MCP server sees every prompt and every tool call that touches it. Treat MCP auth like API keys — rotate, restrict scope, prefer read-only where possible.

### Failure modes

**Server fails to start** — Claude Code logs the error; the tools simply don't appear. Fix the launch command (wrong path, wrong args, wrong env).

**Server crashes mid-session** — subsequent tool calls fail with a clear error. Restart session or reconnect via `/mcp`.

**Auth expires** — if OAuth, re-handshake via `/mcp`. If env-based, update env and restart.

## Why it matters

Claude Code's MCP integration is what makes it more than a code editor. Three practical wins:

1. **Team-wide tool sharing** — committed `.mcp.json` means your teammates clone the repo and immediately have the same GitHub, Jira, database tools available. Zero coordination overhead.
2. **Cross-system workflows** — one Claude Code session can orchestrate across your internal APIs, GitHub, Postgres, Slack — all via MCP servers, all in one loop.
3. **Upgrade path from skills** — prototype a workflow as a skill; if it outgrows Claude Code (needs portability, needs to run from the CLI and from a custom agent), port the tool-call layer to an MCP server and the prompt layer stays a skill.

## When to publish an MCP server

Build an MCP server when:

- You have an external system (DB, internal API, external service) Claude Code should access
- You need it accessible from multiple tools (Claude Code + Cursor + custom SDK)
- The interface is stable enough to version (treat like any service API)

Don't build one when:

- A skill suffices (the work is prompt-shaped)
- You only use it from Claude Code and have no portability needs
- The surface is unstable and changing weekly (skills are easier to iterate)

## Debugging

**"My MCP tool isn't showing up."**
→ `/mcp` first. Server connected? Tools listed? If connected but missing tools, the server's `tools/list` response is the issue.

**"Auth keeps failing."**
→ Check env variables in your shell *before* launching Claude (`echo $GITHUB_TOKEN`). If they're empty, `claude` won't see them either.

**"Server is slow."**
→ MCP adds one round-trip per tool call. Local stdio servers are fast; HTTP-remote servers are as slow as your network. Profile the server.

**"Config changes aren't taking effect."**
→ Most Claude Code versions require restart for MCP config changes. Check the CHANGELOG [4] for any hot-reload additions.

## Key takeaway

**`.mcp.json` is how your repo publishes its MCP toolset.** Commit it — teammates clone, get the same tools. Use the three-scope hierarchy (user / project / local) to separate personal from team from environment-specific. Audit via `/mcp`, debug via env checks, and remember that MCP is a trust boundary worth guarding (Ch 24).

## See Also

- [`15-mcp-protocol.md`](15-mcp-protocol.md) — MCP itself
- [`24-security.md`](24-security.md) — MCP server supply-chain risks
- [`../agents/04-mcp-tools-as-protocol.md`](../agents/04-mcp-tools-as-protocol.md) — General MCP theory

## Sources

[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
[7] Model Context Protocol — <https://modelcontextprotocol.io>
[10] Claude Code MCP docs — <https://code.claude.com/docs/en/mcp>
