# Project: Claude Code Panel — Landing Page

A slim static landing page advertising the **Claude Code — A Primer** session
at the Plano Prompt Engineers meetup. Deployable as static HTML to any host
(GitHub Pages, Netlify, Vercel, a plain S3 bucket).

## Context

- The page serves two roles: marketing the session, and acting as the lab
  target for the panel itself (we demo Claude Code against this codebase
  during the talk).
- Audience is technical developers — the copy assumes they know git and
  basic web tooling.
- Visual style deliberately mirrors the slide deck: warm near-black background
  (`#1a1614`), coral accent (`#d97757`), off-white body text. Keep the palette
  consistent if you add sections.

## Conventions

- **Plain HTML + CSS.** No framework, no build step. Anyone should be able to
  open `index.html` in a browser and see the result. If you introduce
  JavaScript, keep it vanilla and optional — the page must render fully
  without it.
- **Single `styles.css` file.** Don't split into multiple CSS files unless the
  page grows substantially. CSS custom properties (variables) are defined at
  the top of `styles.css`; reuse them instead of hardcoding colors.
- **Semantic HTML.** `<section>` for each top-level block; `<h2>` per section;
  `<ul>` for lists. Avoid `<div>`-soup.
- **Accessible.** Every link has visible text. Contrast ratios on text vs
  background must pass WCAG AA.

## Guardrails

- Do **not** add build tooling (no webpack, no Vite, no package.json) unless
  explicitly asked. Slimness is a feature.
- Do **not** introduce external CSS frameworks (Tailwind, Bootstrap). The
  hand-written `styles.css` is part of the demo surface — changes to it should
  be visible and reviewable.
- Do **not** add analytics, trackers, or third-party scripts.
- Copy that references event details (date, venue, RSVP count) should be
  easy to update — keep it in `index.html` directly, not spread across files.
- If you add a new section, update the "What you'll learn" and "Materials"
  sections' anchors if needed so the page still reads cleanly top-to-bottom.

## Deploying

Push to any static host. Example for GitHub Pages:

```bash
git checkout -b gh-pages
git push -u origin gh-pages
# GitHub Settings → Pages → deploy from gh-pages branch, root
```

## Using this repo as a Claude Code lab

During the panel, we run Claude against this codebase to demonstrate:

- `CLAUDE.md` being read at session start (this file)
- Plan mode for landing-page edits
- Permissions — denying destructive commands, asking on commits
- Headless mode via `claude -p`
- A research-backed lesson via `/tutor --research`

See the parent `README.md` for the panel-level overview.
