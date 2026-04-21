# Claude Code: Install → Configure → Master

A 45-minute hands-on lab.

> **Leave with:** Claude Code installed, a working mental model of its ecosystem, and a real skill (`/tutor`) dropped into your environment.

**Repo:** <https://github.com/debozkurt/claude-intro>

---

## What's in this repo

```
claude-code-panel/
├── presentation.md              # the lab — reads top-to-bottom, or renders to slides
├── presentation.html            # pre-rendered slides (open in browser)
├── presentation.pdf             # pre-rendered handout
├── assets/                      # static resources referenced by the slides
│   ├── linkedin-qr.png
│   └── meetup-qr.png
├── lab-app/                     # slim landing page + live-demo target
│   ├── index.html               # the page itself (Claude-themed)
│   ├── styles.css
│   ├── CLAUDE.md                # project instructions Claude auto-loads at session start
│   ├── README.md
│   ├── docs/
│   │   └── design-tokens.md     # @-referenced from CLAUDE.md — demo of scoped docs
│   └── .claude/
│       ├── settings.json        # checked-in permissions (allow / deny / ask patterns)
│       ├── settings.local.json  # per-user overrides (normally gitignored; checked in here for the demo)
│       └── rules/               # path-scoped, modular instructions — loaded on demand
│           ├── no-build-tooling.md
│           └── html-accessibility.md
├── cheatsheets/
│   ├── commands.md              # CLI + slash commands reference
│   └── permissions.md           # allow/deny/ask patterns
├── claude-code-docs/            # "Master Class" companion guide — 27 chapters + appendices
│   ├── README.md                # index + dependency graph
│   ├── 01-mental-model.md … 27-recipes.md
│   └── appendix-{changelog,cheatsheets,glossary}.md
└── bonus-skills/
    ├── README.md                # install instructions for the take-home artifacts
    ├── tutor/SKILL.md           # the skill walked through in §5 — take it home
    └── tutor-researcher/        # the companion research subagent
```

---

## How to work through this lab

1. **Read `presentation.md` top-to-bottom** (or open `presentation.html` in a browser for slide view).
2. **Copy `lab-app/` to a scratch dir**, `cd` into it, run `claude`:
   ```bash
   cp -r lab-app ~/scratch/claude-lab
   cd ~/scratch/claude-lab
   open index.html          # preview the page
   claude                   # start a Claude session against the repo
   ```
3. **Run the ▶ Try it** steps you find in the presentation against your scratch copy.
4. **Install `bonus-skills/tutor/`** into `~/.claude/skills/tutor/` and invoke `/tutor` on a real problem from your own codebase. See `bonus-skills/README.md` for exact install commands.
5. **When you want to go deeper**, work through `claude-code-docs/` — a self-contained Master Class covering mental models, permissions, plan mode, worktrees, skills, subagents, hooks, MCP, the Agent SDK, orchestration, cost/observability, and a decision tree for picking the right primitive.

---

## Prerequisites

- **Node.js 18+** — `node --version`
- **git** — `git --version`
- **macOS, Linux, or Windows with WSL**
- A Claude account (anthropic.com) **or** an `ANTHROPIC_API_KEY` env var

---

## Install (reference)

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

First run:

```bash
cd <any-project-or-empty-dir>
claude
# inside: /login  (browser auth)
# or set ANTHROPIC_API_KEY before launching
```

Common install hiccups: Node version too old, global npm directory not on `PATH`, or auth not completing — resolve in that order.

---

## License / use

Lab materials — use, adapt, fork freely. The `/tutor` skill is MIT-spirit: copy it, change it, ship it.
