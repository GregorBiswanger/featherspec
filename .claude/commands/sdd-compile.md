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
results. Read the referenced spec, its `.plan.md` sibling if one exists, `AGENTS.md`, and
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
  docs sync clean. Anything else is `NOT READY` — name the blocking items directly after it.
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
- **Do / Don't** — including the *Style & Output Preferences* from `AGENTS.md`.
- **Docs sync** — compare `activeContext.md`'s `Last updated` line against the newest commit in
  the git log above. Check the `architecture:` snapshot and `systemPatterns.md` against
  structural or decision changes in the diff (writes go through the confirmation gate in
  `AGENTS.md`), and that `techContext.md`'s test command matches the one that actually ran.
  If the code moved and a doc did not, name what is missing. Is `activeContext.md` within its
  size limit from `AGENTS.md`? Do the spec and plan statuses match what you just read?
- **Next 3 steps** — concrete and actionable.

Write the brief in `DocLanguage`.
