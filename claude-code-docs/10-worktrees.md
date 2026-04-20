# Chapter 10 — Worktrees with Claude

*Last verified: 2026-04-19 — Prerequisites: Ch 02, Ch 09 — Status: Running Claude*

**Builds on:** [`09-session-lifecycle.md`](09-session-lifecycle.md) · [`../agents/13-when-to-split.md`](../agents/13-when-to-split.md) (when to split work).

---

## Concept

A git worktree is a **separate working directory** attached to the same `.git` object database, with its own branch checked out [git-worktree(1)]. Multiple branches live in parallel folders, not one folder swapping files on `checkout`. Claude Code keys session history per-directory, so worktrees automatically yield isolated Claude sessions — no shared state, no collisions.

The combination (worktrees + session-per-dir) is Claude Code's parallelism primitive. It's how you run multiple Claudes on the same repo without them fighting.

## How it works

### The git primitive

```bash
git worktree add ../feat-auth feature/auth   # create linked dir
git worktree list                            # show all worktrees
git worktree remove ../feat-auth             # clean up
```

Three directories share one `.git` database:

```
~/myrepo/          [branch: main]
~/feat-auth/       [branch: feature/auth]
~/feat-billing/    [branch: feature/billing]
```

All three exist simultaneously. Each has its own files on disk, its own checked-out branch. Commits in one are instantly visible to the others — same object DB.

Constraint: a given branch can only be checked out in *one* worktree at a time. Git enforces this.

### The Claude coupling

Claude Code session state lives at `~/.claude/projects/<project-slug>/` where `<slug>` derives from the absolute working directory [1]. Different worktrees → different slugs → independent session histories.

Practical effect: `claude` in `~/myrepo/` and `claude` in `~/feat-auth/` are as isolated as sessions in totally different repos, despite sharing commits. Each session:

- Has its own conversation JSONL
- Has its own auto-memory (actually, this is shared — memory is keyed by *git repo*, not worktree)
- Can be in a different permission mode
- Sees the same CLAUDE.md hierarchy (repo + user)

Auto-memory exception [1]: worktrees of the same git repo share one auto-memory dir (because auto-memory is keyed by the git repo root, not the working dir). Intentional — what Claude learns about a codebase should persist across the worktree you happened to be in.

## Why it matters

Four workflows unlock:

1. **True parallel Claude** — one session refactoring tests in `../feat-tests/`, another shipping a feature in `../feat-billing/`. No file lock, no merge conflict, no context mix.
2. **Per-worktree risk posture** — throwaway worktree on bypass mode; main worktree on normal mode. Isolation by design.
3. **Safe sandbox** — let Claude go wild in a worktree. If it fails, `git worktree remove` wipes the blast radius. Main repo untouched.
4. **Conversation forking with code-level separation** — Ch 09's `/branch` forks the conversation in the same dir; worktree + fresh `claude` forks both conversation AND code tree.

## Common patterns

**"Two-approach exploration"** — you're weighing approach A vs B. Worktree A with approach A, worktree B with approach B. Two Claude sessions, one per worktree. Compare outcomes. Merge-back the winner.

```bash
git worktree add ../approach-a feat/approach-a
git worktree add ../approach-b feat/approach-b
# two terminals, two claudes, two branches, one shared history
```

**"Risky experiment"** — bypass mode in a worktree, no risk to main:

```bash
git worktree add ../sandbox experiment/yolo
cd ../sandbox
claude --dangerously-skip-permissions
# when done: git worktree remove ../sandbox
```

**"Team-style review"** — your main worktree is your feature work. A second worktree on `main` with a code-reviewer subagent (Ch 19) reviews your commits as they land.

## Merge-back

Worktrees don't "merge." The *branches* in them merge, like any other git branch:

```bash
cd ../feat-auth
git push                              # or commit locally
cd ../main-repo
git merge feature/auth                # or PR on GitHub
git worktree remove ../feat-auth      # clean up
git branch -d feature/auth            # optional
```

Cleanup is two steps: remove the worktree, optionally delete the branch. The worktree's `.git` file (a pointer, not a directory) gets cleaned up automatically by `git worktree remove`.

## Debugging

**"Can't add worktree — branch already checked out."**
→ That branch is live in another worktree. Either detach it (`git worktree remove` the other), or add with `-b` to create a new branch from the target.

**"My two Claude sessions keep talking about the same code."**
→ Check you actually `cd`-ed into the worktree. Same-dir sessions share state.

**"Auto-memory is polluting across worktrees."**
→ Expected — auto-memory is keyed by git repo, not worktree. If you want true isolation, use separate repos (clones), not worktrees.

## Key takeaway

**Worktrees are Claude Code's parallelism primitive.** One repo, N folders, N branches, N Claude sessions — all independent on the session side, all sharing the git object DB on the storage side. When you want two Claudes to work the same repo without collision, the answer is always worktrees.

## See Also

- [`09-session-lifecycle.md`](09-session-lifecycle.md) — `/branch` (conversation fork) vs. worktree (code fork)
- [`20-long-running-claude.md`](20-long-running-claude.md) — Session-per-worktree Ralph patterns
- [`../agents/13-when-to-split.md`](../agents/13-when-to-split.md) — When to parallelize work

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
