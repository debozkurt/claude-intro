# Chapter 15 — MCP: Tools as a Protocol

*Last verified: 2026-04-19 — Prerequisites: Ch 01 — Status: MCP*

**Builds on:** [`../agents/04-mcp-tools-as-protocol.md`](../agents/04-mcp-tools-as-protocol.md) (MCP theory). This chapter is a primer refresher; Ch 16 covers the Claude-Code-specific integration.

---

## Concept

The **Model Context Protocol (MCP)** is an open specification for exposing tools, resources, and prompts to LLM clients over a standard wire protocol [7]. Think of it as the LSP of the agent world: the same MCP server plugs into Claude Code, Cursor, Goose, Warp, and (since 2025) OpenAI and Google DeepMind clients. The protocol, not Anthropic, owns the portability [10].

For a master-class student: understanding MCP matters because it's the answer to "should I build this as a Claude Code skill or as something more portable?" MCP is portable. Skills are Claude Code-specific.

## How it works

### Three primitives the protocol defines

| Primitive | What it exposes | Example |
|---|---|---|
| **Tools** | Callable functions with JSON Schema | `search_jira_issues(query)` |
| **Resources** | Readable content (URIs) | `jira://issue/1234` |
| **Prompts** | Parameterized prompt templates | "summarize-issue" with an `issue_id` arg |

Claude Code (and any MCP client) enumerates these at connection time and treats them like native tools. The LLM sees `search_jira_issues` alongside `Read` and `Bash` — no special handling required.

### The wire

MCP servers communicate over stdio (most common, used for local servers), HTTP (remote), or WebSocket. The protocol is JSON-RPC 2.0 with specific methods (`tools/list`, `tools/call`, `resources/read`, etc.) [7].

### The registry

A vendor-neutral MCP registry launched in 2025 [10]: `registry.modelcontextprotocol.io`. Discover community servers. Quality varies; not all are maintained or safe — see Ch 24 on MCP trust boundaries.

## Why it matters

Three reasons to care:

1. **Portability** — an MCP server you write works across every client that speaks MCP. If you might someday want to run Claude Code, Cursor, *and* a custom agent against the same internal tool (your Jira, your feature flags, your deployment system), MCP is the answer.
2. **Separation of concerns** — the MCP server is just a service exposing tools. It doesn't care about prompt engineering, context, or the LLM. Clean boundary.
3. **Ecosystem access** — hundreds of community servers for GitHub, Slack, databases, Notion, etc. [10] You plug in, you get the tools.

## When MCP beats a skill

Compare MCP servers to Claude Code skills (Ch 12):

| | Skill | MCP server |
|---|---|---|
| Scope | Claude Code only | Any MCP-speaking client |
| Deployment | `SKILL.md` file | Running process / HTTP endpoint |
| Best for | Procedures + prompt patterns | External system integration |
| Tool access | Uses Claude Code's built-in tools | Defines its own tools |

**Use a skill when:** the work is prompt-shaped (procedures, frameworks, multi-step tasks with Claude's existing tools).

**Use MCP when:** you're integrating with an external system (a database, an API, a service) — and you might want portability beyond Claude Code.

## The OpenAI / Google adoption

MCP launched as Anthropic-driven in late 2024. In 2025:
- **OpenAI adopted MCP** for its Assistants / function-calling ecosystem (March 2025)
- **Google DeepMind added MCP support** later in 2025

This matters because MCP is no longer a proprietary integration layer — it's the agent-world's equivalent of HTTP: owned by the spec, implemented by all [7]. Claude Code was one of the earliest mature clients, not the only one.

## Debugging

**"MCP server isn't connecting."**
→ Check the transport: local stdio servers need the process to actually launch (binary on PATH, correct args). Remote HTTP needs the endpoint reachable and auth correct.

**"Tools from the server don't show up."**
→ Run `/mcp` in session; it lists connected servers and their exposed tools. If the server is connected but tools are missing, the server itself isn't exposing them — check its docs.

**"Server is slow."**
→ Tool-call latency dominates many MCP workflows. Profile the server; the wire protocol itself is cheap.

## Key takeaway

**MCP is the portability layer for tools.** Build tools as MCP servers when you want them usable across Claude Code and other clients, or when you're integrating with external systems. Build skills when the work is Claude-shaped and Claude-specific. The two are complements, not competitors.

## See Also

- [`16-mcp-in-claude-code.md`](16-mcp-in-claude-code.md) — Claude Code's specific integration
- [`../agents/04-mcp-tools-as-protocol.md`](../agents/04-mcp-tools-as-protocol.md) — General theory
- [`24-security.md`](24-security.md) — MCP trust boundary

## Sources

[7] Model Context Protocol — <https://modelcontextprotocol.io>
[10] Claude Code MCP docs — <https://code.claude.com/docs/en/mcp>
