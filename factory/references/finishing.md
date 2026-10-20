# Phase 4 — Finish

The build is done, reviewed, and committed locally on the feature branch. Now hand the
decision of what happens to it back to the human (Gate 3).

## Summarise first

Preface the menu with a short summary:

- What was built (the change, in a line or two).
- The branch name and that all commits are local.
- Any findings **carried** past the fix-loop cap, and any deferred minors — the human
  sees these before choosing.

## The menu

Present exactly these options and **wait**:

1. **Merge to the base branch locally** — merge the feature branch into the branch
   `/factory` started from (its BASE, e.g. `main-dupe`), not `main`, unless the human
   says otherwise. No remote action.
2. **Push and open a PR** — push the branch and open a pull request. This is the
   **only** path that touches a remote. The commits and the PR description carry **no
   Claude attribution** (no `Co-Authored-By: Claude`, no "Generated with Claude Code").
3. **Leave the branch as-is** — do nothing further; the work sits on the local branch
   for the human to handle.

Do not pick for them, and do not take any remote action unless they choose option 2.
