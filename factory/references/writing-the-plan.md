# Phase 2 — Write the plan

Turn the approved spec into an implementation plan an engineer (or a builder subagent)
could execute without having seen the discovery. The plan records the decisions they
can't make alone: which files, which names and signatures, which values from the spec,
which test proves each task. Then hold it for approval (Gate 2).

## Header

Start the plan with:

- **Goal** — one sentence.
- **Architecture** — 2–3 sentences on the approach.
- **Spec** — path to the approved spec (the plan argues from it; they travel together).
- **Global Constraints** — the spec's project-wide requirements, one line each with
  exact values (version floors, naming/copy rules, "no remote git on its own", "no
  Claude attribution", verification commands).
- **Review Focus** — the handful of inputs or failure modes the spec implies but no
  task's tests exercise, most-likely-to-bite first. Each gets its test added to the
  task that owns the code.

## Tasks

Break the work into tasks. A task is the smallest unit that carries its own test cycle
and is worth a fresh review — fold setup/config/scaffolding into the task whose
deliverable needs them; split only where a reviewer could reject one and approve its
neighbour. Each task ends with an independently testable deliverable.

Each task has:

- **Files** — exact paths to create / modify (with line ranges) / test.
- **Interfaces** — what it consumes from earlier tasks (exact signatures) and produces
  for later ones (exact names, parameter and return types).
- **Steps**, in RED→GREEN→commit order, each one action with a checkable result:
  1. Write the failing test (its name and assertions, with the spec's exact values).
  2. Run it, confirm it fails for the expected reason.
  3. Implement the minimum — give the exact signature and the file; the builder writes
     the body. A code block only for an algorithm the signature and test don't
     determine.
  4. Run it, confirm it passes.
  5. Commit (locally; no Claude attribution).

Keep the plan to the decisions the builder can't make alone. A line that decides
nothing ("handle edge cases", "add validation") is a gap; a function body the
signature and tests already determine is a transcript. Avoid both.

## Self-review

With the spec in front of you, check: every spec requirement maps to a task; every
step lets the builder write exactly one reasonable thing; types/signatures named in
later tasks match earlier ones; each Review Focus item has a test in its owning task;
the plan isn't several times longer than the spec. Fix issues inline.

## Gate 2

Present the plan and **wait for the human to approve it** before any code is written.
`/factory` adds this gate on top of a bare request — an approved spec does not flow
into the build on its own.
