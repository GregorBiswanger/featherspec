---
description: Final readiness check — verdict, evidence per acceptance criterion, tests, docs sync.
argument-hint: "[path-to-spec.md] [runTests:true|false]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-compile workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the one-line loader in
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
`.memory-bank/activeContext.md`.

**Re-derive every verdict from the repository, not from the plan's checkboxes.** Read the code
and the tests before the plan's claims about them, and treat a ticked box as a claim to check.
If you implemented this work in this same session, say so in one line at the top of the brief —
the context that produced a gap is the worst placed to find it.

## What counts as evidence

Evidence is a **test name plus its pass/fail output**, a **command plus its output**, or a
`file:line`. A step ID is not evidence. Prose describing the implementation is not evidence.
A criterion with no machine artifact behind it is `pending`, never `satisfied` — if a person
checked it by hand, write `pending (manual check by <who>, <what they looked at>)`.

## Produce a concise readiness brief

- **Verdict** — one of `READY` · `NOT READY` · `NOT READY — unverified`. It is
  `NOT READY — unverified` whenever the test suite did not run, whatever the criteria say.
  Name the blocking items directly after it.
- **Current goal** — from the referenced spec.
- **Constraints** — from `AGENTS.md` and the spec.
- **Architecture snapshot highlights** — relevant parts of the `architecture:` block.
- **Acceptance criteria** — each one, marked satisfied / pending, with evidence as defined above.
- **Plan state** — open vs. finished steps, and whether the traceability table names real code
  paths and a test per criterion. Flag any criterion no step covers, and any finished step whose
  `Verified:` field is empty.
- **Do / Don't** — including the *Style & Output Preferences* from `AGENTS.md`.
- **Docs sync** — compare `activeContext.md`'s `Last updated` line against the newest commit in
  the git log above. If the code moved and the Memory Bank did not, say so and name what is
  missing. Do the spec and plan statuses match what you just read?
- **Next 3 steps** — concrete and actionable.

Write the brief in `DocLanguage`.
