---
name: factory
description: Use when taking a feature, bug, or change from a rough one-line request all the way to reviewed, locally-committed code; when you would otherwise hand-run discovery, spec, plan, build, and review yourself; when you want multi-agent review on a change before it lands. Triggers include "/factory", "build this end to end", "run the full pipeline on this".
---

# Factory

## Overview

`/factory <what you want>` drives one change from a rough request to reviewed,
locally-committed code. It does not invent its own process — it **orchestrates**
the superpowers skills for the generic phases and adds its own build-and-review
pipeline for the parts that differ. It stops at three human gates and never
performs a remote git action on its own.

**Core principle:** make the good process (grounded discovery, a written spec,
a plan, multi-dimension review) the default path, while the decisions that are
the human's to make stay the human's.

## The flow

```
/factory <description>
      │
      ▼
1. DISCOVERY + GROUNDED BRAINSTORM + SPEC   → superpowers:brainstorming
      │
   ┌──────────────┐
   │ HUMAN GATE 1 │  approve the spec — nothing is built before this
   └──────────────┘        (this IS brainstorming's hard-gate)
      │
      ▼
2. PLAN                                      → superpowers:writing-plans
      │
   ┌──────────────┐
   │ HUMAN GATE 2 │  approve the plan + execution approach before any code
   └──────────────┘
      │
      ▼
3. BUILD PIPELINE                            → references/build-pipeline.md
      local branch · resumable opus builder · parallel read-only reviewers
      · fix-loop (max 2 rounds, adjudicate on cap-out) · final approver
      │
   ┌──────────────┐
   │ HUMAN GATE 3 │  finishing menu: merge locally / push+PR / leave as-is
   └──────────────┘        (this IS finishing-a-development-branch)
```

## Phases

**1. Discovery, grounded brainstorm, spec.**
**REQUIRED SUB-SKILL:** superpowers:brainstorming. Run it fully — the
loop-until-clear discovery back-and-forth, producing the written design doc. Ground
it in evidence: **dispatch read-only codebase exploration** (the Explore agent, or
`codebase-memory` where available) to gather the affected files, prior art, and
conventions the spec must account for — findings feed the spec, guesses are marked as
guesses. Its hard-gate (nothing built until the human approves the doc) **is Gate 1**.
Do not shortcut discovery even when the request looks obvious.

**How to pose the questions.** This adapts brainstorming's discovery for `/factory`:
same loop-until-clear depth, but **structured**, not one prose question at a time.

- Use Claude's **`AskUserQuestion`** tool rather than free prose whenever the decision
  has recognisable options. Give each question **2–4 concrete, mutually-exclusive
  options**, each with a one-line description of what it means or trades off. The
  human can always pick "Other" to free-type — you don't add that yourself.
- **Batch independent questions** — up to **four per `AskUserQuestion` call** — so the
  human answers a round together instead of a message per question. Group what's
  genuinely independent; keep a follow-up that depends on an answer for the next round.
- Put your **recommended option first** and say why in its description.
- Set `multiSelect` when choices aren't exclusive (e.g. "which fields to export").
  The 2–4 options cap still applies — if there are more candidates than fit, offer
  the common few and let "Other" cover the rest.
- Reserve **open free-text** questions for genuinely open discovery where fixed
  options would railroad the answer (the real "why", unknown constraints).
- Keep looping rounds until the need is clear — structured questions speed each round;
  they don't cap how many you ask.

**2. Plan.**
**REQUIRED SUB-SKILL:** superpowers:writing-plans, off the approved spec. Then
present the plan to the human and **wait** — this is **Gate 2**. `/factory` adds
this gate on top of stock: an approved spec does not flow into the build.

**3. Build pipeline.**
Read `references/build-pipeline.md` and follow it: create a local branch, dispatch
the resumable builder, fan out the read-only reviewers the change needs, run the
capped fix-loop, then the final approver.

**4. Finish.**
**REQUIRED SUB-SKILL:** superpowers:finishing-a-development-branch. Present its menu
and **wait** — this is **Gate 3**. Preface it with a summary of what was built and
any findings carried past the fix-loop cap. No remote action happens unless the
human picks the push+PR option.

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

When run here in isuzu-intranet, this picks up actions-not-services, Form Requests,
`tw-` Tailwind, Spatie DTOs, the `vendor/bin/pint --dirty` / `vendor/bin/phpstan` /
`php artisan test --compact` / `npm run lint` verification, and the
business-rules-docs sync — all automatically, none of it baked into the skill.
