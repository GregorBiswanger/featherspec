---
description: Turn a spec into a persisted baby-step plan file with research and traceability.
argument-hint: "[path to spec or plan]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-plan workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the one-line loader in
     .github/prompts/sdd-plan.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-plan — Implementation Plan

The user may name a spec or plan path after the command. If none is given, list what sits under
`.specs/` and ask which one to work on.

Planning produces a **file**. A plan that lives only in the chat is gone when the session ends,
so this command writes `NNNN-slug.plan.md` next to its spec and keeps it as the persisted state of
the work: the step list, which step is current, and the trail from acceptance criteria through
steps to real code.

## Pick the mode

| Situation | Mode |
| --- | --- |
| The spec has no plan file yet | **A — plan from scratch** |
| A plan exists and work is unfinished | **B — resume** |
| A plan exists but the spec changed | **C — re-plan the delta** |

## Mode A — plan from scratch

1. **Read** the spec, `AGENTS.md` (constraints, `architecture:` snapshot, style preferences),
   and `.memory-bank/techContext.md` plus `systemPatterns.md`.
2. **Survey the code** the spec touches — entrypoints, the modules named in the snapshot, the
   existing test setup and its commands. Plan against the repo as it is, not as it should be.
   Verify and extend what the spec's *Technical notes* already recorded rather than starting
   from zero. **If the survey contradicts the spec** — a criterion assumes behaviour the code
   does not have, or ignores a caller it would break — stop, report the contradiction as a spec
   defect, and switch to Mode C. Do not plan around a spec you have just refuted.
3. **Research** what you would otherwise guess (see below).
4. **Decompose into baby steps** (see below).
5. **Write the plan file**, then hand it over for review and stop (see *Always* below).

## Research (do it, do not skip it)

Plan against current facts, not recollection. Use your web search or fetch capability whenever
the spec involves a library, framework or API you cannot verify from the repo · version-specific
behaviour · a protocol, standard or regulation · a pattern where your knowledge may be stale.

- Check the version actually used in the repo (lock file, manifest) before trusting a doc page.
- Prefer official documentation, release notes, and the project's own repository.
- Record every source under `## Research`: title, link, one line on what it settled, and the
  date you retrieved it. No link, no claim.
- If no web access is available, say so plainly and mark the affected steps as assumptions
  rather than presenting a guess as fact.

## Baby steps

A step is one focused change that can be finished and checked on its own:

- **One concern per step** — a schema change, one endpoint, one component, one test suite.
- **Small enough** to complete in one sitting and to read in one diff.
- **Verifiable**: every step carries a `Verify:` line — a command whose output decides the step
  (test, build, lint, script). Only where the domain genuinely has no machine check — visual
  layout, wording, a third-party sandbox — write `manual: <what a person looks at>` and say why
  no command can settle it. If you cannot state either, the step is too big or too vague; split it.
- **Recorded**: the step's `Verified:` field stays empty until the `Verify:` line was actually
  run. A tick without a recorded result is a claim, not a verification.
- **Ordered so the repo keeps working** after every step; risky or blocking parts come first.
- **Tied to the spec**: each step names the acceptance criteria it serves, every criterion is
  covered by at least one step, and pure scaffolding steps say so explicitly.

If the step list runs long, the spec was probably two specs. Say so before writing the file.

## Plan file structure

Restated here because path-scoped rules load when a matching file is **read**, and a brand-new
plan has not been read yet:

- Write the plan in `DocLanguage`.
- Name it after its spec with a `.plan.md` suffix, in the **same lifecycle folder**:
  `0007-user-login.md` → `0007-user-login.plan.md`.
- Status line near the top, vocabulary `Not started | In Progress | Blocked | Done`.

````markdown
# Plan — <spec title>

**Spec:** [0007-user-login.md](0007-user-login.md)
**Status:** Not started
**Last updated:** <date>
**Current step:** T-001

## Approach

Two or three sentences: the strategy, why the steps are ordered this way, and which
`.memory-bank/*` files the implementation will touch.

## Research

- [Title](https://example.org/doc) — what it settled, retrieved <date>

## Steps

### T-001 — <short imperative title>

- [ ] **Covers:** AC-001, AC-002
- **Do:** what changes, in which files or modules
- **Verify:** the command whose output decides this step (or `manual: …` plus the reason)
- **Verified:** _(empty until it was actually run: date · command · result)_
- **Notes:** _(filled while implementing: deviations, findings)_

### T-002 — <short imperative title>

- [ ] **Covers:** AC-003
- **Do:** …
- **Verify:** …
- **Verified:** —
- **Notes:** —

## Traceability

| Acceptance criterion | Steps | Code / files | Test | State |
| --- | --- | --- | --- | --- |
| AC-001 | T-001, T-004 | _(filled when the step lands)_ | _(the test that fails without the code)_ | open |

`State` is one of `open | built | verified`. A criterion is `built` when the code exists and
`verified` only when a recorded run proves it. If no test can decide it, write
`manual: <who checked what>` in the `Test` cell — an empty cell means nobody checked.

## Session handoff

- **Done so far:** —
- **Next action:** T-001
- **Open decisions:** —
- **Environment:** build, run and test commands needed to continue
````

## Mode B — resume an existing plan

1. Read the plan first, then the spec. `Current step` and `Session handoff` say where the work
   stands — verify that against `git status --short` and the actual code before trusting it.
2. Report in three lines: what is done, what is next, what blocks it.
3. Continue from the next open step only when the user asks you to. After each finished step,
   update the plan **in the same change set**: tick the box, fill `Notes`, write the real paths
   into the traceability table, move `Current step`, refresh `Last updated` and the handoff.
4. When every step is ticked and its criteria hold, set the plan status to `Done` and point the
   user at `/sdd-compile`.

## Mode C — the spec changed

1. Compare spec and plan: which acceptance criteria are new, changed, or gone?
2. Read the traceability table **in reverse** — for every touched criterion, list the steps and
   the code paths already built from it. That list *is* the impact: the code a change to this
   requirement reaches.
3. Report the impact before editing anything: criterion → steps → files, plus what becomes
   obsolete.
4. Then extend the plan: append new steps, **never renumber** existing IDs (they are references),
   strike obsolete steps with a one-line reason instead of deleting them, and set the plan status
   back to `In Progress`.

## Always

- Write the plan and every report in `DocLanguage`.
- **Planning does not change code, and the plan is not yours to approve.** In Mode A and C, stop
  after writing the file and hand it to the user: name the plan, say how many steps it has, name
  the riskiest one, and say plainly that reading these steps now is cheaper than reading the diff
  later — a wrong step costs hundreds of lines, a wrong line costs one. Ask which steps look
  wrong before anything is implemented. Every other artifact here has a named reader; this one
  is the most expensive to get wrong.
- Your own todo or task list is scratch state that dies with the session. The plan file is the
  durable one — when the two differ, the file wins and gets corrected.
- Keep the plan lean — it is a working document, not a design essay. Requirements belong in the
  spec, long-lived decisions in `.memory-bank/systemPatterns.md`.
