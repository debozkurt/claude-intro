# Claude Code: Install → Configure → Master

A 45-minute hands-on lab.

> **Leave with:** Claude Code installed, a working mental model of its ecosystem, and a real skill (`/tutor`) dropped into your environment.

---

## What's in this repo

```
claude-code-panel/
├── presentation.md              # the lab — reads top-to-bottom, or renders to slides
├── presentation.html            # pre-rendered slides (open in browser)
├── presentation.pdf             # pre-rendered handout
├── lab-app/                     # slim landing page + live-demo target
│   ├── index.html               # the page itself (Claude-themed)
│   ├── styles.css
│   ├── CLAUDE.md                # project instructions Claude loads
│   └── .claude/settings.json    # permissions
├── cheatsheets/
│   ├── commands.md              # CLI + slash commands reference
│   └── permissions.md           # allow/deny/ask patterns
└── bonus-skills/
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
4. **Install `bonus-skills/tutor/`** into `~/.claude/skills/tutor/` and invoke `/tutor` on a real problem from your own codebase.

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
