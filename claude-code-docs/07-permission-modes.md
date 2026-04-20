# Chapter 07 — Permission Modes in Depth

*Last verified: 2026-04-19 — Prerequisites: Ch 01 — Status: Running Claude*

**Builds on:** [`../agents/18-human-in-the-loop.md`](../agents/18-human-in-the-loop.md) (Human-in-the-loop).

---

## Concept

Claude Code's permission system is a **four-mode, allow/deny/ask** overlay on every tool call [1]. The primer introduced the four modes; this chapter goes into *why* they exist in this exact shape, *when* to reach for each, and how the trust model composes with `settings.json`.

The core claim: permissions aren't a UX gate — they're the main security boundary between Claude and your system. Get the model wrong and a skill can exfiltrate your `.env`; get it right and you can safely run Claude on `main`.

## How it works

### The four modes

| UI label | Config name | Auto-approves | Still asks |
|---|---|---|---|
| **Normal** | `default` | Nothing | Every tool call |
| **Auto-accept edits** | `acceptEdits` | `Read`, `Edit`, `Write` | `Bash`, `Agent`, `WebFetch`, etc. |
| **Plan** | `plan` | Nothing (read-only planning) | Nothing — no state-changing tools run |
| **Bypass permissions** | `bypassPermissions` | Everything | Nothing |

Shift+Tab cycles through Normal → Auto-accept edits → Plan. `bypassPermissions` is deliberately outside the cycle — you have to pass `--dangerously-skip-permissions` on the command line.

### The `settings.json` overlay

On top of the mode, the `permissions` block in `settings.json` provides per-tool-pattern control [1]:

```json
{
  "permissions": {
    "allow": ["Read", "Edit", "Bash(npm test:*)", "Bash(git status:*)"],
    "deny":  ["Bash(rm -rf*)", "Bash(curl:*)", "Bash(sudo:*)"],
    "ask":   ["Bash(git commit:*)", "Bash(git push:*)"]
  }
}
```

Precedence: **deny > ask > allow > mode default**. A matched `deny` blocks the tool with no prompt, no recovery. A matched `ask` prompts regardless of mode (even in bypass). `allow` silently approves. Anything unmatched falls through to the mode's default.

Glob matching: `:*` after a command matches any args. `"Bash(git status)"` matches exactly `git status`; `"Bash(git status:*)"` matches any `git status` variant.

### Three layers of settings

1. `~/.claude/settings.json` — user defaults
2. `<repo>/.claude/settings.json` — team settings (committed)
3. `<repo>/.claude/settings.local.json` — personal project overrides (gitignored)

Arrays are **unioned** across layers — a `deny` from any layer wins. Non-array keys: later layer overrides earlier.

## Why it matters

The four modes map to four distinct risk postures:

- **Normal** — first contact with unfamiliar code or untrusted repos
- **Auto-accept edits** — flow state on a project you trust; edits fly, but shell is still gated
- **Plan** — review-first discipline for non-trivial changes (Ch 08)
- **Bypass** — sandboxed/throwaway environments only; the permission system is entirely off

Intermediate users often sit in Normal and churn through prompts. Mastering Claude Code means actively moving between modes to match the moment. A long editing session is Auto-accept. A delicate refactor starts in Plan. Normal becomes the default at risk boundaries (touching `production/`, writing Dockerfiles, etc.).

## Practical patterns

**The safe-Bash starter allowlist** (see `cheatsheets/permissions.md` in the primer repo):

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Edit", "Write",
              "Bash(ls:*)", "Bash(cat:*)", "Bash(pwd)",
              "Bash(git status:*)", "Bash(git diff:*)", "Bash(git log:*)",
              "Bash(npm test:*)", "Bash(npm run:*)", "Bash(node:*)"],
    "deny":  ["Bash(rm -rf*)", "Bash(sudo:*)", "Bash(curl:*)", "Bash(wget:*)",
              "Bash(git push --force*)", "Bash(git reset --hard*)"],
    "ask":   ["Bash(git commit:*)", "Bash(git push:*)", "Bash(npm install:*)"]
  }
}
```

Reading and testing are free. Destruction is denied. State-changing git asks once.

**The iterative tuning pattern** — run for a day, note which prompts feel like friction (add to `allow`), which felt scary (add to `deny` or tighten to `ask`). Permissions grow to match your habits [2].

**Ask Claude to write it** — concrete prompt:

> "Review my shell history and this repo's stack. Draft a `.claude/settings.json` permissions block that allows routine dev commands, denies destructive ones, and asks before state-changing git. Explain each line."

Claude knows its own permission syntax well; this is one of the few places where "let the agent write the config" actually works.

## Security implications

Don't forget [2][8]:

- **`allow` is a capability grant** — every entry in `allow` is something Claude can do without your consent. Treat it like an IAM policy.
- **`deny` is the only hard stop** — not mode-dependent, not mood-dependent. Deny `rm -rf*` even in projects you trust.
- **Bypass mode is not "trust Claude more"** — it's "turn off the safety system for this session." Only in sandboxes.
- **Settings.local.json is for loosening, never for weakening denies** — your personal overrides can `allow` more, but should rarely (never) `deny` less than the team baseline.

See Ch 24 for the full security treatment including prompt injection.

## Key takeaway

**Permissions are the main security boundary.** Use the four modes actively — don't sit in Normal by default. Configure `settings.json` around your actual workflow (ask Claude to help). And remember: `deny > ask > allow > mode default` is a hard rule, not a suggestion.

## See Also

- [`08-plan-mode.md`](08-plan-mode.md) — Plan mode in depth
- [`14-hooks.md`](14-hooks.md) — Hooks as a second layer of deterministic control
- [`24-security.md`](24-security.md) — Prompt injection, MCP trust, YOLO blast radius
- [`../agents/18-human-in-the-loop.md`](../agents/18-human-in-the-loop.md) — General HITL patterns

## Sources

[1] Claude Code Docs — permissions — <https://code.claude.com/docs/en/iam>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
