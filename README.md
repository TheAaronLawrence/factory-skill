# factory

A [Claude Code](https://claude.com/claude-code) skill that drives one change —
a feature, a bug, a tweak — from a rough one-line request all the way to
reviewed, locally-committed code, through a multi-agent pipeline that stops at
the decisions that are genuinely yours to make.

Type `/factory <what you want>` and it runs:

```
/factory <description>
      │
      ▼
1. DISCOVERY + GROUNDED BRAINSTORM + SPEC   → superpowers:brainstorming
      │
   ┌──────────────┐
   │ HUMAN GATE 1 │  approve the spec — nothing is built before this
   └──────────────┘
      │
      ▼
2. PLAN                                      → superpowers:writing-plans
      │
   ┌──────────────┐
   │ HUMAN GATE 2 │  approve the plan before any code
   └──────────────┘
      │
      ▼
3. BUILD PIPELINE
      local branch · resumable builder · parallel read-only reviewers
      · fix-loop (max 2 rounds, adjudicate on cap-out) · final approver
      │
   ┌──────────────┐
   │ HUMAN GATE 3 │  finishing menu: merge locally / push+PR / leave as-is
   └──────────────┘
```

## What makes it different

- **Three human gates, and only three.** It never writes code before the spec
  and plan are approved, and never touches a remote (push, PR) on its own — that
  only happens if you pick it at the finishing menu. Everything in between runs
  without pestering you.
- **Discovery that asks well.** The questions round uses structured
  multiple-choice (`AskUserQuestion`) — batched, recommended option first, with
  open free-text reserved for the real "why".
- **Multi-agent review, scoped to the change.** A builder implements the plan;
  read-only reviewers (code + security always, UX + UI-design when the change
  touches UI) run in parallel; a capped fix-loop resolves findings; a final
  approver takes one whole-branch pass.
- **Portable.** It discovers each project's own conventions (`CLAUDE.md`,
  `.claude/rules/`) and review agents (`.claude/agents/`) at runtime, falling
  back to general-purpose reviewers where a project has none. Nothing about any
  one repo is baked in.

## Install

Copy the `factory/` directory into your Claude Code skills directory:

```bash
git clone https://github.com/TheAaronLawrence/factory-skill.git
cp -R factory-skill/factory ~/.claude/skills/factory
```

Then, in any project, invoke it:

```
/factory add a CSV export to the contacts page
```

It's user-scoped, so `/factory` is available in every project you work in.

## How it's built

`/factory` orchestrates the [superpowers](https://github.com/obra/superpowers)
skills for the generic phases (brainstorming, writing-plans,
finishing-a-development-branch) and carries its own build-and-review pipeline in
`factory/references/build-pipeline.md`. The subagent prompts live in
`factory/references/builder-brief.md` and `factory/references/reviewer-brief.md`.

## License

MIT — see [LICENSE](LICENSE).
