---
paths:
  - .memory-bank/activeContext.*.md
---

# Parallel work on one plan

Applies whenever a plan step is implemented outside the main session — by a subagent, a
second session, another tool or model, or an agent in a git worktree. One rule keeps all of
them conflict-free: **the main session is the only writer of the plan, `activeContext.md`
and git history; a worker writes only its own step's files and one return note.**

## Starting a parallel stage (main session)

- Only steps the plan joins with `‖` qualify, each with a `Files:` line. Run them in sequence
  unless the user chose otherwise; no delegation in your tool means in sequence.
- Before the first worker starts, note the stage base — the HEAD commit and the
  `git status --short` output; integration compares against it.
- **Subagents:** delegate each step to `sdd-implementer` with the plan path and the step ID,
  at most three at a time. A subagent cannot ask: where a working agreement in `AGENTS.md`
  wants a stop it cannot make — a test review, say — do that part yourself first (write the
  stage's tests, get them confirmed) and delegate only the implementation, with the confirmed
  tests named as off-limits.
- **Other agents:** print one ready-to-paste start line per step —
  `Implement T-00x of <plan path> as a parallel worker.` — and wait until the user says they
  are back. A worktree holds committed files only — no uncommitted work, no ignored files
  such as installed dependencies: before printing the lines, propose the commit that brings
  the tree level (Ask first), name it as the stage base, and say that a worktree must be
  created from it. A worker in its own session keeps the working agreements with its user —
  a test review happens there.

## Worker contract

- **Check your base first.** What your step builds on — the files of the earlier stages,
  their ticks in the plan — must be in your checkout. Missing (a worktree created before
  the main tree was committed): stop and say what is needed — the main session commits,
  then that commit is merged into your worktree (a git write: ask). Never rebuild earlier
  steps, and never write outside your checkout — not into the main tree next door either.
  Installing dependencies a fresh worktree lacks is Ask-first.
- Implement exactly the one step you were given — its `Do:`, test first where it adds
  behaviour and no confirmed test was named — and touch only the files its `Files:` line
  names. Need another file: stop.
- Run only the step's own `Verify:`. Other workers share the tree, so the *Quality gate*
  agreement in `AGENTS.md` is met by the main session: it runs the gates on the combined
  result before any step is ticked.
- Never edit the plan or any Memory Bank file other than your return note, never tick a box,
  make no git write. The one exception: in your own worktree the step's files and the note
  go back as one commit on your branch — propose it (Ask first); the main session merges
  that branch.
- An Ask-first action, a `Verify:` that stays red, a confirmed test that would have to
  change, or a question only the main session can answer: stop and report `BLOCKED`. A
  subagent cannot ask anyone; a worker in its own session may ask its user what a working
  agreement requires, nothing more.
- Last act: write the return note; your final message is its content and nothing else.

## Return note

`.memory-bank/activeContext.<spec number>-<step ID>.md` — e.g. `activeContext.0007-T-003.md`
— at most 15 lines, in `DocLanguage`:

```markdown
# Return note — 0007 · T-003

- **Status:** DONE | BLOCKED
- **Files:** every file created or changed
- **Verified:** date · command · what was red → the green result
- **Notes:** deviations from `Do:`, findings — or —
- **Blocked on:** the question or Ask-first action (only when BLOCKED)
```

A return note lying in `.memory-bank/` means: parallel work came back and is not integrated.

## Integrating (main session only)

1. The changes since the stage base are exactly the union of the steps' `Files:` plus the
   notes, and each note's `Files:` stays within its own step's line — `git status --short`,
   or `git diff --name-only <stage base>..HEAD` once work from a worktree has arrived by
   merge (the merge is a git write: Ask first). Anything else is a finding to raise before
   going on.
2. Re-run each step's `Verify:` yourself, then the plan's quality gates on the combined tree.
   A worker's claim is not evidence until you have seen it pass.
3. Only then tick the steps with their `Verified:` results, carry `Notes` over, sync as at
   any stop, and delete the integrated notes — the one deletion `AGENTS.md` exempts from
   Ask-first.
4. `BLOCKED` is a stop: report what the worker needs; the other steps' results stand. Its
   note stays until the step is finished — by you, or by a restarted worker whose note
   replaces it — and is then integrated and deleted like the others.
