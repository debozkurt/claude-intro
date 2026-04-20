# Chapter 04 — `.claude/rules/`: Path-Scoped Modular Instructions

*Last verified: 2026-04-19 — Prerequisites: Ch 03 — Status: Context & Memory*

**Builds on:** [`03-claude-md-discipline.md`](03-claude-md-discipline.md). **Fresh ground** for path-scoping mechanics.

---

## Concept

`.claude/rules/` is the release valve for CLAUDE.md overflow. It's a directory of markdown files — each one a focused ruleset on a single topic — with optional YAML `paths:` frontmatter that controls *when* the rule loads [1]. The design answers two problems at once: splitting a bloated CLAUDE.md into maintainable pieces, and scoping rules to only load when Claude is actually working with relevant files (context savings, especially in monorepos).

## How it works

### Directory layout

```
<repo>/
├── CLAUDE.md                         # lean — just what's always needed
└── .claude/rules/
    ├── code-style.md                 # loaded every session (no paths:)
    ├── testing.md                    # loaded every session
    ├── api-conventions.md            # path-scoped (below)
    └── frontend/
        └── react-patterns.md         # nested, path-scoped
```

All `.md` files under `.claude/rules/` are discovered recursively. User-level rules live at `~/.claude/rules/` and apply to every project.

### Unconditional vs path-scoped

**Without frontmatter** — loaded at session start, same priority as `.claude/CLAUDE.md`:

```markdown
# Testing conventions

Tests use Vitest. Run with `npm test`. New files mirror source layout.
```

**With `paths:` frontmatter** — loaded only when Claude reads a matching file:

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/handlers/**/*.ts"
---

# API conventions

All endpoints validate input via zod. Errors follow the standard shape.
```

Glob patterns follow `.gitignore`-style rules [1]. Loading is triggered when Claude *reads* a matching file — not on every tool use.

### Symlinks for sharing

`.claude/rules/` supports symlinks. Maintain one authoritative rules directory and link it into multiple repos:

```bash
ln -s ~/company-standards .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

Symlinks are resolved and loaded normally. Circular links are detected.

## Why it matters

Three wins over stuffing everything in CLAUDE.md:

1. **Context savings in monorepos** — your frontend team's 300 lines of React conventions only load when Claude touches `src/frontend/`. Backend sessions get backend-only rules.
2. **Modular maintenance** — each file is small, single-topic, separately git-reviewable. Bus-factor-friendly.
3. **Shared rules across repos** — symlink company standards in; changes propagate via filesystem, not copy-paste.

Rule of thumb [2]: if CLAUDE.md is over ~200 lines, or if you catch yourself writing "only when working on X" disclaimers in CLAUDE.md, move those rules to `.claude/rules/` with appropriate `paths:`.

## What to put where

| Content | Home |
|---|---|
| Always-relevant project conventions | `CLAUDE.md` |
| Large multi-topic rule sets | `.claude/rules/<topic>.md` (no paths) |
| Rules that apply to one subdir or filetype | `.claude/rules/<topic>.md` (with paths:) |
| Company-wide standards | `~/company-standards/` symlinked into each repo |
| Your personal style preferences | `~/.claude/rules/preferences.md` |

## Debugging

**"My rule isn't loading."**
→ Run `/memory` in session — it lists *all* loaded CLAUDE.md and rule files. If your rule isn't there, check: (a) filename ends in `.md`, (b) file is under `.claude/rules/` (not elsewhere), (c) if path-scoped, you've actually opened a matching file this session.

**"Path-scoped rule loaded too aggressively."**
→ Your glob is too broad. `**/*.ts` matches every TypeScript file. Narrow to the directory: `src/api/**/*.ts`.

**"Rule conflicts with CLAUDE.md."**
→ Both load; more specific *and* more recently-loaded wins. Path-scoped rules are loaded when a matching file is opened, so they effectively "win" late in the turn. Usually desired.

## Key takeaway

**`.claude/rules/` is how mature Claude Code projects stay disciplined.** Once CLAUDE.md passes ~200 lines or you need rules that only apply to subfolders, split into path-scoped rule files. Symlinks let you share standards across repos without copy-paste.

## See Also

- [`03-claude-md-discipline.md`](03-claude-md-discipline.md) — When to split CLAUDE.md
- [`06-context-cache.md`](06-context-cache.md) — How rules interact with the prompt cache
- [`12-skills-in-depth.md`](12-skills-in-depth.md) — When to move from rule to skill

## Sources

[1] Claude Code Docs — memory — <https://code.claude.com/docs/en/memory>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
