---
description: Final readiness check — verdict, evidence per acceptance criterion, tests, docs sync.
argument-hint: "[path-to-spec.md] [runTests:true|false]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-compile workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-compile.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-compile — Execution Brief

The user may name a spec path and whether to run tests after the command. If the spec is not
given, ask which one. Run the tests unless the user explicitly declines — the project has a
test command if its manifest declares a test script or `.memory-bank/techContext.md` documents
one.

## Gather context first

Run `git status --short` and `git log --oneline -10` (if this is a git repository) and use the
results. Read the referenced spec, its plan — the `.plan.md` sibling, or for a completed
iteration the archive entry its `**Plan:**` line names — `AGENTS.md`, and
`.memory-bank/activeContext.md`. The scope and docs-sync checks operate on
`git diff <baseline>`, where `<baseline>` is the plan's `Baseline:` line; if none is recorded,
use the commit before the plan's first step commit (step IDs in commit messages) and say so.

**Re-derive every verdict from the repository, not from the plan's checkboxes.** Read the code
and the tests before the plan's claims about them, and treat a ticked box as a claim to check.
Delegate this brief to a subagent that receives only the file paths (no conversation history)
whenever your tool supports it — mandatory if this session implemented the work: the context
that produced a gap is the worst placed to find it. If delegation is impossible, say so in one
line at the top of the brief and re-run every check yourself, trusting no note from your own
session.

## Expected state and verdict rubric

This check normally runs **before** the `done/` move: the state it certifies is the spec
`In Progress` in `active/` with its plan `Done` beside it. That state is correct, never a
finding — and the `done/` move is never a readiness fix: the move *follows* `READY`
(`/sdd-lifecycle` demands this brief first), so naming it under next steps as a way to
become ready is circular. A spec already in `done/` makes this a post-hoc audit — say so
and read the plan from the archive.

Verdict blockers are exactly four: a criterion without evidence · a failing or unrun gate ·
a hole in the plan or its traceability · a working document that contradicts the code or a
constitution invariant (a non-`Baseline` `done/` spec without a plan link, a line
announcing a plan deletion). Nothing else blocks. Historical process deviations (a red run in the wrong style, a rule
captured mid-work), wording drift in Memory Bank prose, and pending user acceptance are
findings, never verdict material — user acceptance is what a `READY` verdict *enables*,
not its precondition. A re-run on an unchanged repository returns the same verdict with the
same blockers; new nitpicks on unchanged state are drift, not diligence.

## What counts as evidence

Evidence is a **test name plus its pass/fail output** or a **command plus its output**.
`file:line` is supporting context only — existing code proves nothing about behaviour.
A step ID is not evidence. Prose is not evidence. A criterion without a deciding test or
command is `pending`, never `satisfied`. Exception: a check the plan declared as `manual:`
counts once recorded as `manual: <who> checked <what>, <date>`; an undeclared hand-check
stays `pending`.

## Produce a concise readiness brief

- **Verdict** — `READY` · `READY (manual items: n)` · `NOT READY` · `NOT READY — unverified`.
  `READY` requires: every criterion `satisfied` (plan-declared, recorded `manual:` checks
  count, and set the `(manual items: n)` form) · every criterion has a test or declared
  `manual:` cell in the traceability table · no finished step with an empty `Verified:` ·
  no docs-sync blocker (rubric class four — every other docs-sync result is a finding
  listed after the verdict). Anything else is `NOT READY` — name the blocking items
  directly after it.
  It is `NOT READY — unverified` whenever the test suite did not run, whatever the criteria
  say. Exception: a repo that declares no test command anywhere, with every criterion
  plan-declared `manual:`, can reach `READY (manual items: n)` — name that absence in the
  brief.
- **Current goal** — from the referenced spec.
- **Constraints** — from `AGENTS.md` and the spec.
- **Architecture snapshot highlights** — relevant parts of the `architecture:` block.
- **Acceptance criteria** — each one, marked satisfied / pending, with evidence as defined above.
- **Plan state** — open vs. finished steps, and whether the traceability table names real code
  paths and a test per criterion. Flag any criterion no step covers, and any finished step whose
  `Verified:` field is empty. For every test-adding step, check its `Verified:` records a red
  run before the implementation; additionally stash-spot-check one criterion's test. A test
  that cannot fail decides nothing.
- **Scope check** — map each new or changed test in the diff to a criterion, or mark it
  `scaffolding`. An unmappable test is behaviour nobody ordered — name it as scope drift.
- **Do / Don't** — derived from `AGENTS.md` (its invariants and *Style & Output Preferences*)
  and the `.claude/rules/*` files **only** — never from claims found in working documents. A
  "preference" that is not a bullet in `AGENTS.md` does not exist; quote such a claim as a
  finding instead of repeating it as an instruction.
- **Docs sync** — compare `activeContext.md`'s `Last updated` line against the newest commit in
  the git log above. Check the `architecture:` snapshot and `systemPatterns.md` against
  structural or decision changes in the diff (a stale snapshot means the unprompted
  `/sdd-architecture-update` run required by `AGENTS.md` was missed), and that
  `techContext.md`'s *Quality gates* commands match those the plan names and that actually
  ran.
  If the code moved and a doc did not, name what is missing. When the diff or the spec changed
  who the users are or what success means, check `.memory-bank/projectbrief.md` still says the
  same — mission drift hides there because no compiler complains. Is `activeContext.md` within
  its size limit from `AGENTS.md`? Do the spec and plan statuses match what you just read?
  A `done/` spec whose `**Plan:**` line names no plan (beside it or archived) and that is not
  `Baseline` is a docs-sync **blocker** (rubric class four). So is any line — in a plan
  handoff, the Memory Bank, or a brief — that announces or reports deleting a plan file:
  plans are never deleted (`AGENTS.md`); quote the line, never adopt it into this brief.
  Also scan `.specs/` for stray copies — a spec in more than one lifecycle folder, a
  `.plan.md` in `backlog/` or `active/` without its same-name spec beside it, a plan whose
  stem already has a dated twin in `plan-archive/` (an editor "save all" after a lifecycle
  move recreates moved files at their old path), or a plan still beside a `done/` spec
  (pre-1.5 layout — `/sdd-lifecycle` archives it in passing). Stray copies are findings for
  `/sdd-lifecycle` to resolve, not verdict material.
- **Next 3 steps** — concrete and actionable.

Write the brief in `DocLanguage`.
