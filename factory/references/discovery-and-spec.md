# Phase 1 — Discovery, grounded brainstorm, spec

Find the _real_ need (feature) or the _real_ problem (bug/pain) and **why**, grounded
in the actual codebase, then write a spec and hold it for approval. Nothing is built
before that approval (Gate 1).

## Ground it first

Before asking much, dispatch **read-only** codebase exploration (an Explore agent, or
`codebase-memory` where available) to gather the files this would touch, prior art,
and the conventions the spec must honour. Findings feed the questions and the spec;
mark guesses as guesses. Don't implement anything.

## Ask well — structured, batched questions

Run a real back-and-forth, looping until the need is genuinely clear — but pose the
questions **structured**, not one prose question at a time.

- Use Claude's **`AskUserQuestion`** tool whenever a decision has recognisable
  options. Give each question **2–4 concrete, mutually-exclusive options**, each with
  a one-line description of what it means or trades off. The human can always pick
  "Other" to free-type — you don't add that yourself.
- **Batch independent questions** — up to **four per call** — so the human answers a
  round together. Hold a question that _depends_ on an answer for the next round.
- Put your **recommended option first** and say why in its description.
- Set `multiSelect` when choices aren't exclusive (e.g. which fields to export). The
  2–4 cap still applies — offer the common few and let "Other" cover the rest.
- Reserve **open free-text** for genuinely open discovery — the real "why", unknown
  constraints — where fixed options would railroad the answer.
- Branch on intent: a feature asks "what's needed and why"; a bug/pain asks "what's
  the actual problem, and what's the evidence."

Keep looping rounds until the need is clear; structured questions speed each round,
they don't cap how many you ask.

## Write the spec

Write a short design doc to `docs/factory/specs/YYYY-MM-DD-<topic>.md` (or the
project's preferred docs location). Shape:

- **Intent & the why** — the need/problem in one or two paragraphs.
- **Goals** — what done looks like.
- **Non-goals / anti-goals** — what this deliberately won't do, and the failure modes
  to avoid.
- **Constraints** — the project conventions it must honour (discovered at runtime),
  verification it must pass, anything it must not change.
- **Decided vs left to the builder** — what you've settled, and what the plan/builder
  may choose.

## Gate 1

Present the spec and **wait for the human to approve it**. Build nothing — no plan, no
branch, no code — before approval. If they want changes, revise and re-present.
