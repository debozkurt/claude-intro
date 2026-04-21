# No build tooling

This rule is always-on (no `paths:` frontmatter) — it applies to every
session in this repo, regardless of which file Claude is editing.

**The rule:** do not introduce a build step to this project.

- No `package.json`, no `node_modules/`, no lockfiles.
- No bundlers (webpack, Vite, esbuild, Parcel, Rollup).
- No CSS frameworks or preprocessors (Tailwind, Bootstrap, Sass, PostCSS).
- No TypeScript. Plain `.js` only if JS is strictly needed.

**Why:** slimness is the demo surface. The point of `lab-app/` is that a
viewer can `open index.html` in any browser and see the result — zero
install, zero ceremony. A build step breaks that promise.

**If you think you need one, stop and ask.** The answer is almost always
"inline it as vanilla CSS/HTML instead."
