# lab-app

A slim static landing page for the **Claude Code — A Primer** session at the Plano Prompt Engineers meetup. Doubles as the lab target for live demos during the panel.

## Run locally

```bash
cd lab-app
open index.html          # macOS
# or: xdg-open index.html   (Linux)
# or: start index.html      (Windows)
```

Pure HTML + CSS — no build step, no dependencies.

## Use as a Claude Code lab

```bash
cd lab-app
claude
```

You're now in a Claude session on a real project. Suggested things to try:

- Ask: *"What does this project do? What's the visual style?"* — watch `CLAUDE.md` land.
- `Shift+Tab` to plan mode. Ask: *"Add a 'Prerequisites' section with code blocks."* See the plan before it executes.
- In a second terminal: `claude -p "list every section heading in index.html"`
- Try a denied command: *"run `rm -rf node_modules`"* — watch the permission deny.
- `/tutor --research "CSS contrast ratios for accessibility"` — trigger the research subagent; get a citation-grounded lesson persisted to your docs.

## Files

- `index.html` — the page itself
- `styles.css` — Claude-themed palette (dark warm bg + coral accent)
- `CLAUDE.md` — project instructions Claude loads every session
- `.claude/settings.json` — permissions (allow routine edits, deny destructive)

## Deploying

Any static host. GitHub Pages, Netlify drop, Vercel, S3. Nothing to build.
