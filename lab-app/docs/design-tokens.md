# Design tokens — Claude Code Panel landing page

The palette deliberately mirrors the slide deck. Reuse these tokens instead
of hardcoding hex values. All tokens are defined as CSS custom properties at
the top of `styles.css`.

## Palette

| Token              | Hex       | Use                                        |
|--------------------|-----------|--------------------------------------------|
| `--bg`             | `#1a1614` | Page background (warm near-black)          |
| `--bg-lead`        | `#141210` | Lead/hero background                       |
| `--accent`         | `#d97757` | Claude coral — headings, links, CTAs       |
| `--accent-soft`    | `#f0c9b6` | Inline code foreground                     |
| `--text`           | `#e8e6e3` | Body text (off-white)                      |
| `--text-strong`    | `#f5f1e8` | `<strong>` + emphasized text               |
| `--text-muted`     | `#8a857d` | `.small`, captions, pagination             |

## Contrast

All text/background pairs must pass **WCAG AA** (4.5:1 for body, 3:1 for
large text). The palette above is pre-checked — if you introduce a new color,
run it through a contrast checker before merging.

## Typography

- System font stack: `-apple-system, BlinkMacSystemFont, "Segoe UI", Inter, sans-serif`
- Body size: `26px`
- Line height: `1.45`
- Headings: `600` weight, tight letter-spacing (`-0.02em`) on h1

## When to deviate

Don't. If a new section feels like it needs a new color, the section
probably belongs on a different page.
