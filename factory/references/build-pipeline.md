# Factory build pipeline

This is Phase 3 of `/factory`, entered only **after** the plan is approved at Gate 2.
It runs the build on a local branch, reviews it with the agents the change needs,
loops on fixes within a hard cap, then takes one final whole-branch pass. No remote
git action happens here — the finishing menu (Gate 3) owns that.

Follow the sections in order.

## Create the branch

- Record the starting branch as **BASE** (e.g. `main-dupe`). The finishing menu's
  local-merge target is BASE, not `main`, unless the human says otherwise.
- Create a local feature branch off BASE, named from the topic
  (`<type>/<kebab-topic>`, e.g. `feature/dealer-notes`). All commits are local.

## Dispatch the builder

Dispatch **one** builder as a resumable subagent and keep its handle for the
fix-loop:

- Agent/Task tool, `subagent_type: general-purpose` (it needs full tools), **model
  opus**, and a **stable `name`** — `factory-builder` — so the fix-loop can
  `SendMessage` the same agent instead of re-priming a fresh one.
- Prompt = `references/builder-brief.md` with its `{{...}}` slots filled from the
  approved plan and the conventions discovered for this project (see
  SKILL.md → Runtime discovery).
- Let it implement the whole plan task-by-task and commit locally per task. When it
  reports done, move to the reviewer fan-out.
- If the builder agent cannot be resumed later (handle lost, context gone), re-prime
  a fresh `factory-builder` with the plan plus the **complete** findings list — never
  a partial one.

## Reviewer fan-out

Once the builder reports a buildable branch, dispatch the reviewers the change needs
**in a single message** (so they run in parallel) and **read-only** (each gets
`references/reviewer-brief.md`, which forbids edits/HEAD-moves/subagents).

**Which reviewers:**

- **Always:** `code` (sonnet) and `security` (opus).
- **Only if the change touches UI:** `ux` (sonnet) and `ui-design` (sonnet). A
  pure-backend change skips them — do not dispatch an empty review.

**UI-detection heuristic.** A change "touches UI" if any file in `git diff
--name-only {{base}}..HEAD` defines or styles something the user sees — a component,
a template/view, or a stylesheet. Decide portably, not by one stack's paths:

- **Extensions (any stack):** `.vue`, `.jsx`, `.tsx`, `.svelte`, `.astro`,
  `.blade.php`, `.twig`, `.erb`, `.html`, and any stylesheet (`.css`, `.scss`,
  `.sass`, `.less`, `.styl`).
- **Locations:** any path under a directory the discovered conventions name as the
  UI/front-end root. If the conventions don't name one, infer from the project's
  layout — e.g. `resources/js/` + `resources/views/` (Laravel), `src/components/`,
  `app/`, `pages/`, `components/` (JS frameworks). When unsure whether a changed file
  is user-facing, **treat it as UI** — an extra review is cheaper than a silent skip.

Otherwise (pure data/logic/config/tests) it does not touch UI, and ux + ui-design are
skipped. Runtime discovery (SKILL.md) is where you learn this project's UI root; do
not assume a Laravel layout.

**Which agent fills each role:**

- If the project has a matching agent in `.claude/agents/` (e.g. `code-reviewer`,
  `security-reviewer`), dispatch it as-is — it is already read-only.
- Otherwise (ux, ui-design, or a project with no such agent) dispatch a
  `general-purpose` agent on the role's model with `reviewer-brief.md`.

**Collect verdicts.** Gather each reviewer's `Ready to merge?` line and graded
findings. Verdicts can disagree: treat the **union of all Critical and Important
findings** as the blocking set — a single "No"/"With fixes" means there is work to
do, regardless of another reviewer's "Yes". Pass that set to the fix-loop.

## Fix-loop

Run at most **two** fix→re-review rounds. The loop must always exit — never a third
round, never a silently dropped finding.

A round:

1. If the blocking set (union of Critical + Important) is empty, the loop is done →
   go to the final approver.
2. Resume the builder (`SendMessage` to `factory-builder`) with the **complete**
   blocking set — every finding, with reviewer, grade, and `file:line`.
3. When it reports its fixes, re-run **only the reviewers whose findings were in the
   set** (scoped re-review), read-only, in one message. Recompute the blocking set
   from their new verdicts.
4. Rounds used +1. If a finding remains and you are **under 2 rounds**, loop. If you
   have **used 2 rounds**, stop and adjudicate.

**Adjudicate on cap-out.** For each finding still open after round 2, make a ruling:

- **Contestable or not load-bearing** (style disagreement, a risk that cannot occur
  on this change): park it with a one-line ruling of why it stands.
- **Load-bearing** (a reasonable user gets a broken or unsafe result): if a bounded
  fix is clear, apply it via the builder once more as part of adjudication; if not,
  **carry** it — and it MUST appear in the Gate 3 summary so the human sees it before
  choosing what to do with the branch.

**Minor** findings are recorded in the summary and never enter the loop.

Why the cap is 2 (not the agent's instinctive 3+): the diagram fixes it at 2, and a
bounded loop plus explicit adjudication reaches a human decision faster than extra
silent rounds. "Keep going until clean" is not a licence for an unbounded loop.

## Final approver

Once the fix-loop has exited, dispatch **one** final reviewer over the whole branch
(`git diff {{base}}..HEAD`), read-only, on **opus** (the strongest model — this is the
judgment call that decides the branch is sound). Give it the spec, the plan, and the
list of rulings made during the fix-loop so it can weigh them.

Its Critical/Important findings get **one** fix pass through the builder (each fixed
with TDD, then the verification commands re-run). Minor findings join the Gate 3
summary. After that single pass, the build phase is done — proceed to Gate 3. Do not
reopen the fix-loop.

## Models

The reference diagram fixes the model per role. Dispatch each agent on the model
below (set the Agent/Task `model` explicitly; an omitted model inherits the
session's, which may not match).

| Role                    | Model  |
| ----------------------- | ------ |
| spec-writer             | opus   |
| builder                 | opus   |
| security reviewer       | opus   |
| code reviewer           | sonnet |
| ux reviewer             | sonnet |
| ui-design reviewer      | sonnet |
| final review / approver | opus   |
