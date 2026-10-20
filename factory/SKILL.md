---
name: factory
description: Use when taking a feature, bug, or change from a rough one-line request all the way to reviewed, locally-committed code; when you would otherwise hand-run discovery, spec, plan, build, and review yourself; when you want multi-agent review on a change before it lands. Triggers include "/factory", "build this end to end", "run the full pipeline on this".
---

# Factory

## Overview

`/factory <what you want>` drives one change from a rough request to reviewed,
locally-committed code. It carries its **own** process end to end — discovery, spec,
plan, a multi-agent build-and-review pipeline, and a finishing menu — stopping at
three human gates and never performing a remote git action on its own.

The approach is **inspired by** the superpowers skills and Anthropic's skill-authoring
guidance, but `/factory` is **self-contained**: every phase's method lives in this
skill's own `references/` files. It does not invoke those skills at runtime.

**Core principle:** make the good process (grounded discovery, a written spec, a plan,
multi-dimension review) the default path, while the decisions that are the human's to
make stay the human's.

## The flow

```
/factory <description>
      │
      ▼
1. DISCOVERY + GROUNDED BRAINSTORM + SPEC   → references/discovery-and-spec.md
      │
   ┌──────────────┐
   │ HUMAN GATE 1 │  approve the spec — nothing is built before this
   └──────────────┘
      │
      ▼
2. PLAN                                      → references/writing-the-plan.md
      │
   ┌──────────────┐
   │ HUMAN GATE 2 │  approve the plan before any code
   └──────────────┘
      │
      ▼
3. BUILD PIPELINE                            → references/build-pipeline.md
      local branch · resumable builder · parallel read-only reviewers
      · fix-loop (max 2 rounds, adjudicate on cap-out) · final approver
      │
   ┌──────────────┐
   │ HUMAN GATE 3 │  finishing menu: merge locally / push+PR / leave as-is
   └──────────────┘                         → references/finishing.md
```

## Phases

**1. Discovery, grounded brainstorm, spec.** Follow `references/discovery-and-spec.md`:
ground the request in the real codebase (read-only exploration), run a looping
back-and-forth posed as **structured, batched `AskUserQuestion` rounds**, then write
the spec. Its approval is **Gate 1**. Do not shortcut discovery even when the request
looks obvious.

**2. Plan.** Follow `references/writing-the-plan.md`: turn the approved spec into a
task-and-step plan with exact files, interfaces, and RED→GREEN→commit steps. Present
it and **wait** — this is **Gate 2**. An approved spec does not flow into the build on
its own.

**3. Build pipeline.** Follow `references/build-pipeline.md`: create a local branch,
dispatch the resumable builder, fan out the read-only reviewers the change needs, run
the capped fix-loop, then the final approver.

**4. Finish.** Follow `references/finishing.md`: summarise what was built (and any
carried findings), then present the finishing menu and **wait** — this is **Gate 3**.
No remote action happens unless the human picks the push+PR option.

## Rules that must hold

**Violating the letter of these rules is violating their spirit.**

**1. The three gates are hard stops. `/factory` never writes code before Gate 1 and
Gate 2 have both been approved by the human.** Discovery → spec → _approval_ →
plan → _approval_ → build, in that order, every time. A one-line request is the
_input_ to discovery, never a licence to skip it. Urgency, an authorising ticket,
and "the lead said ship it" change nothing: the gates exist precisely for the
moments you feel pressure to skip them.

| Rationalization (seen in testing)                      | Reality                                                                                                                                                                             |
| ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "The ticket says go straight to code / just commit it" | The ticket is the request `/factory` runs _through_ its gates, not around them.                                                                                                     |
| "The lead already authorised committing"               | They authorised the outcome, not skipping the spec and plan. The gates are how you reach a _correct_ outcome.                                                                       |
| "Don't overthink it — the request is obvious"          | Baseline agents who skipped discovery shipped broken, out-of-scope code (a call to a model that did not exist). "Obvious" is where grounded discovery is cheapest and most skipped. |
| "We're late, the gates cost time"                      | Re-doing work that skipped the spec costs more. The gate is a pause for the human, not a delay you remove.                                                                          |
| "I'll write the spec after, once the code works"       | A spec written to match code already written is a description, not a decision. It cannot gate anything.                                                                             |

**Red flags — STOP, you are about to skip a gate:**

- You are reading or editing implementation files and no spec has been approved.
- You are about to create a branch or dispatch the builder and no plan has been approved.
- You are telling yourself the request is too simple or too urgent for discovery.

All of these mean: stop, return to the phase you skipped, and run it.

**2. No remote git on `/factory`'s own initiative.** The build creates a local
branch and local commits only. Push, PR, and any other remote/shared action happen
**only** when the human picks that option at Gate 3. Commits and PRs carry **no
Claude attribution** (no `Co-Authored-By: Claude`, no "Generated with Claude Code").

**3. Reviewers are read-only.** They report findings; they never edit the tree, move
HEAD, or spawn subagents. This is enforced in `references/reviewer-brief.md`; keep it
true for every reviewer you dispatch, including fallback general-purpose ones.

## Runtime discovery

`/factory` is user-scoped and runs in any project, so it never hardcodes one repo's
rules. At the start of the build phase, discover the active project and pass what you
find into the builder and reviewer briefs:

- **Conventions.** Read the project's `CLAUDE.md` (and `AGENTS.md` if present) and
  every file in `.claude/rules/`. Inline them into the builder's `{{conventions}}`
  and each reviewer's `{{conventions}}` slot — the agents judge against the project's
  own rules, not assumptions.
- **Reviewer agents.** List `.claude/agents/`. For each role, reuse a matching
  project agent (`code-reviewer`, `security-reviewer`, …) if it exists; otherwise
  dispatch a `general-purpose` agent under `references/reviewer-brief.md` on that
  role's model. A project with no `.claude/agents/` at all still gets a full review
  via the fallback.
- **Verification commands.** Use whatever the conventions specify as the build's
  verification (tests, formatter, linter, static analysis). Only run commands the
  project actually defines — do not assume a stack.
- **Business-rules-docs sync.** Only when the project has a rule requiring it (e.g. a
  `business-rules-docs` rule) does the builder update doc pages in the same change.
  Projects without it skip this entirely.

For example, in a Laravel + Vue project whose `CLAUDE.md` mandates actions-not-services,
Form Requests, `tw-`-prefixed Tailwind, and Spatie DTOs, with verification
`vendor/bin/pint --dirty` / `vendor/bin/phpstan` / `php artisan test` / `npm run lint`,
all of that is picked up automatically — none of it is baked into the skill.
