---
paths:
  - "**/*.html"
---

# HTML accessibility

This rule is path-scoped — it only loads when Claude touches an `.html`
file. For CSS-only or docs edits it stays out of context, saving tokens.

**Required when editing HTML:**

- Use semantic tags: `<section>`, `<header>`, `<nav>`, `<main>`, `<footer>`.
  No `<div>`-soup.
- Every `<h*>` is numbered in order — skipping levels breaks screen readers.
- Every `<img>` has a non-empty `alt=""` attribute (or `alt=""` if purely
  decorative, explicitly).
- Every `<a>` has visible text — no bare URLs, no `<a>` wrapping only an icon.
- Color contrast between text and background must pass WCAG AA (4.5:1 body,
  3:1 large text). Use tokens from `docs/design-tokens.md`.

**Why path-scoping:** the accessibility rules don't matter when Claude is
editing CSS tokens or docs. Loading them only on HTML keeps CLAUDE.md's
base instructions lean for every other kind of task.
