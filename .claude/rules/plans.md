---
paths:
  - .specs/**/*.plan.md
---

# Plan file rules

The plan artifact itself — one plan per spec, the `.plan.md` naming, the status vocabulary, and
the duty to keep it current — is defined in `AGENTS.md` (loaded every session). These rules
cover only what is specific to **working inside a plan file**:

- Write in the language specified by `DocLanguage` in `AGENTS.md`.
- Steps stay baby steps: one behaviour slice — the test and the code for the criteria that
  the same tests decide and the same code serves, typically three to five — finishable in
  one sitting, with a `Verify:` line that is a command whose output decides it. Never split
  by layer or file: a stub, a type or a test without its code lands in the step that needs
  it. A step you cannot verify on its own is a step to split; one past six criteria or
  about 150 changed lines is a split candidate.
  Where no machine check exists, `Verify:` reads `manual: <what a person looks at>` plus the
  reason no command can settle it.
- **Tick the checkbox only after `Verified:` is filled with a real result** — date, the command
  you ran, and what it returned. An empty `Verified:` means the step is not done, however
  finished the code looks. This is the one rule that keeps the plan from becoming a record of
  intentions.
- A test-adding step runs its test red before the implementation lands; `Verified:` then
  records both runs in one entry — *what* was red (failing assertion vs. missing module) and
  the green result. A test that was never red decides nothing. Name the AC ID in the test's
  name or description, so the mapping survives outside the plan. Red-first binds new
  behaviour only; a preserved-behaviour criterion is decided by its existing test staying green.
- **Per finished step, one edit, in the same change set as the code:** tick the box and fill
  `Verified:` — plus `Notes:` only when you deviated from `Do:` or found something a later
  step needs. That edit is the durable record of progress; nothing else is written per step.
  Keep turns few: send independent tool calls together — reads, edits to different files.
- **When a stage ends** — also when you are blocked, when the user pauses the work, or
  before you report "done" — bring the rest level in one pass: the traceability rows of the finished
  steps with real paths (`src/auth/login.ts`, or `file:symbol`) and the test that decides each
  criterion, `State` moving `open → built → verified` only on a recorded run; `Current step`;
  `Last updated`; and `Session handoff`, rewritten so a fresh session can continue from the
  file alone. Never write a path that does not exist yet — that is what `Do:` is for.
  Resuming after an interruption, the ticks and the code lead over `Current step` and the
  handoff: bring those level first.
- **One step, then stop.** Work the next open step to its end — its `Verify:` and the
  quality gates green, the one edit above — and stop there: two lines on what was built and
  what comes next, and one commit proposal carrying the step ID. Never start the next step
  unasked, not even its test. A step that closes its stage (`## Stages`) brings the sync
  above before the stop. More in one go only when the user asks for it — "the whole stage",
  "the whole plan": then the steps follow each other, and the stop is the end of that stage
  or, for the whole plan, the end of the plan; every stage end on the way still brings the
  sync above. Stop at once, either way, when a `Verify:` stays red, an Ask-first action
  comes up or a deviation would change behaviour. A plan without `## Stages` treats every
  step as its own stage.
- **A finished plan ends the work here.** With the last step ticked, set `Status: Done`, bring
  the sync, and recommend `/sdd-compile` — the user's own command, in a new session, best with
  a strong model from another model family than the one that implemented: a reviewer that
  shares the author's blind spots finds less. Never run that check, or the `/sdd-lifecycle`
  move after it, on a "continue".
- **A stop that a working agreement in `AGENTS.md` asks for is never skipped, moved or
  collected in advance** — whatever the user asked to run in one go. The test review is the
  common one: write the new or changed test, run it red, show the cases as written, and
  stop. Only the user's yes to those tests, given after they exist, lets you implement. A
  plan approval, a start signal, an answer to another question or expectations described
  beforehand are not that yes; do not bundle the question with other approvals.
- Steps joined by `‖` are independent and may run in parallel. Ask once how to run them —
  at the start signal when the plan has a `‖` stage, at the latest before the first one —
  and keep the answer for the plan's later `‖` stages. Name what each way changes: **in
  sequence** (the default: nothing changes); **subagents** (they cannot ask you — you write
  the stage's tests yourself and have them confirmed before delegating, and there is no
  steering in between); **other agents** (you paste one start line per step into a second session,
  another tool or model, or a worktree — a worktree needs the tree committed first). Say
  too that either parallel way the quality gates run once, on the combined result, before
  any step is ticked, and recommend one: in sequence when the steps are small — parallel
  pays for large slices. Where the tool has a question picker, use it. Read
  `.claude/rules/parallel-work.md` before you start such a stage.
- **One session writes a plan.** You are a parallel worker — read
  `.claude/rules/parallel-work.md` at once, and edit neither this file nor
  `activeContext.md` — when you were started with `… as a parallel worker`, or when the
  user says another session is working on this plan. Asked for a step whose earlier steps
  are still open in your checkout — typical in a fresh worktree — ask once which you are
  before doing anything. Re-cutting the stages of a plan in progress belongs to the main
  session, at a stage end.
- Record deviations in the step's `Notes:`. A plan that quietly drifts from the code is worse
  than no plan.
- Keep step IDs stable. They are referenced from commits, reports, and the traceability table;
  append new ones instead of renumbering, and strike obsolete steps with a reason.
- Research entries keep their link and retrieval date, so a later session can tell fresh
  findings from stale ones.
- It stays a working document, not a diary: current state and the decisions that still matter.
- An archived plan (`.specs/plan-archive/NNNN-slug.YYYY-MM-DD.plan.md`) is **frozen**: never
  edit, extend, renumber or delete it (the single closing edit `/sdd-lifecycle` makes while
  archiving is part of the freeze, not an exception). What a later iteration learns belongs
  in that iteration's own plan; the spec's `## Plan history` line carries the pointer.
  Deleting a plan file is never sanctioned — `AGENTS.md` owns that invariant.

> Note: path-scoped rules load when a matching file is **read**. When `/sdd-plan` creates a
> brand-new plan, this rule is not loaded yet — which is why that command restates the
> essentials (DocLanguage, the naming, the status vocabulary, the file skeleton) inline. That
> duplication is deliberate; do not "clean it up". `AGENTS.md` stays authoritative.
