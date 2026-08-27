---
paths:
  - .specs/**/*.plan.md
---

# Plan file rules

The plan artifact itself — one plan per spec, the `.plan.md` naming, the status vocabulary, and
the duty to keep it current — is defined in `AGENTS.md` (loaded every session). These rules
cover only what is specific to **working inside a plan file**:

- Write in the language specified by `DocLanguage` in `AGENTS.md`.
- Steps stay baby steps: one concern, finishable in one sitting, with a `Verify:` line that is
  a command whose output decides it. A step you cannot verify on its own is a step to split;
  as a rule of thumb a step covers one or two acceptance criteria — more than three is a
  split candidate.
  Where no machine check exists, `Verify:` reads `manual: <what a person looks at>` plus the
  reason no command can settle it.
- **Tick the checkbox only after `Verified:` is filled with a real result** — date, the command
  you ran, and what it returned. An empty `Verified:` means the step is not done, however
  finished the code looks. This is the one rule that keeps the plan from becoming a record of
  intentions.
- A test-adding step records its failing run first; the passing run lands with the
  implementation. A test that was never red decides nothing. Name the AC ID in the test's
  name or description, so the mapping survives outside the plan — and say in `Verified:`
  *what* was red (failing assertion vs. missing module). Red-first binds new behaviour only;
  a preserved-behaviour criterion is decided by its existing test staying green.
- **Update the file the moment a step's state changes**, in the same change set as the code:
  tick the checkbox, fill `Verified:` and `Notes`, move `Current step`, refresh `Last updated`,
  and rewrite `Session handoff` so a fresh session can continue from the file alone.
- **Fill the traceability table with real paths** (`src/auth/login.ts`, or `file:symbol`) as
  soon as a step lands, and name the test that decides the criterion in the `Test` column.
  Never write a path that does not exist yet — that is what `Do:` is for. `State` moves
  `open → built → verified`; it reaches `verified` only on a recorded run.
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
