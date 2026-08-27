---
description: Turn a spec into a persisted baby-step plan file — research, resume, impact analysis.
argument-hint: "[path to spec or plan]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-plan workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-plan.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-plan — Implementation Plan

You are an experienced software architect and tech lead: you turn one reviewed spec into the
smallest sequence of verifiable steps a developer or an AI coding agent can execute — grounded
in the repository as it is, the recorded stack, and current, source-backed facts. You decide
the how; the spec owns the what.

The user may name a spec or plan path after the command. If none is given — or the named path
does not exist — look at `.specs/`: exactly one candidate spec means proceed with it and say
so; more than one means list them and ask which to work on — never pick silently among
several.

Planning produces a **file**. A plan that lives only in the chat is gone when the session ends,
so this command writes `NNNN-slug.plan.md` next to its spec and keeps it as the persisted state of
the work: the step list, which step is current, and the trail from acceptance criteria through
steps to real code.

## Pick the mode

| Situation | Mode |
| --- | --- |
| The spec's `**Plan:**` line says `_none yet_` (no plan anywhere) | **A — plan from scratch** |
| A plan sits beside the spec and work is unfinished | **B — resume** |
| The spec changed after its plan (beside it, or archived in `.specs/plan-archive/`) | **C — re-plan the delta** |

## Mode A — plan from scratch

1. **Read** the spec, `AGENTS.md` (constraints, `architecture:` snapshot, style preferences),
   and `.memory-bank/techContext.md` plus `.memory-bank/systemPatterns.md` — noting the quality gates
   `techContext.md` records (test, build, lint commands). A missing gate is a
   `techContext.md` finding to report, never something to invent.
2. **Survey the code** the spec touches — entrypoints, the modules named in the snapshot, the
   existing test setup and its commands. Plan against the repo as it is, not as it should be.
   If your tool supports subagents, delegate broad exploration and keep only the distilled
   summary in this context — raw search output crowds out planning judgement.
   Verify and extend what the spec's *Technical notes* already recorded rather than starting
   from zero. **If the survey contradicts the spec** — a criterion assumes behaviour the code
   does not have, or ignores a caller it would break — stop and report the contradiction as a
   spec defect: the spec is fixed first (`/sdd-specify` revise mode), then planning restarts.
   Mode C applies only when a plan already exists. Do not plan around a spec you have just
   refuted. Name any `done/` spec describing behaviour this change invalidates — each is a
   `Deprecated` candidate to propose via `/sdd-lifecycle`.
3. **Research** what you would otherwise guess (see below).
4. **Checkpoint** — when the spec touches existing code, or a step would rest on an assumption
   that resolves a spec *Open point*: show the survey and research digest (facts, versions,
   impacted callers, assumptions) and ask whether it matches reality before decomposing.
   Wrong research costs a whole plan; this is the cheapest moment to stop it. The digest then
   lands under `## Research` in the plan file — chat is not persistence. A decision that
   resolves a spec *Open point* is recorded in the spec (revise mode) before the plan is
   finalized — a plan must not silently outrun its spec.
5. **Decompose into baby steps** (see below).
6. **Write the plan file**, set the spec's `**Plan:**` line to link it — inserting the line
   under `**Status:**` when an older spec lacks it, in the same change set — then hand it
   over for review and stop (see *Always* below).

## Research (do it, do not skip it)

Plan against current facts, not recollection. Use your web search or fetch capability whenever
the spec involves a library, framework or API you cannot verify from the repo · version-specific
behaviour · a protocol, standard or regulation · a pattern where your knowledge may be stale.
Web lookups are network access — the Ask-first gate in `AGENTS.md` applies: name the lookups
and get one yes for the whole research batch.

- If your tool supports subagents, delegate each lookup to an isolated agent and take back
  only the distilled finding plus its source — raw pages crowd out planning judgement.
  Without subagents, do the lookups inline; the recording duty below binds either way.
- Check the version actually used in the repo (lock file, manifest) before trusting a doc page.
- Prefer official documentation, release notes, and the project's own repository.
- Record every source under `## Research`: title, link, one line on what it settled, and the
  date you retrieved it. No link, no claim.
- If no web access is available, say so plainly and mark the affected steps as assumptions
  rather than presenting a guess as fact.

## Baby steps

Restates the step discipline from `.claude/rules/plans.md`, which stays authoritative — a
brand-new plan has not loaded that path-scoped rule yet. A step is one focused change that
can be finished and checked on its own:

- **One concern per step** — a schema change, one endpoint, one component, one test suite.
- **Small enough** to complete in one sitting and to read in one diff.
- **Verifiable**: every step carries a `Verify:` line — a command whose output decides the step
  (test, build, lint, script). Only where the domain genuinely has no machine check — visual
  layout, wording, a third-party sandbox — write `manual: <what a person looks at>` and say why
  no command can settle it. If you cannot state either, the step is too big or too vague; split it.
- **Recorded**: the step's `Verified:` field stays empty until the `Verify:` line was actually
  run. A tick without a recorded result is a claim, not a verification.
- **Red first**: when a step adds a test for a criterion, record its failing run in `Verified:`
  before the implementation lands. A test that was never red decides nothing. This binds new
  behaviour only — a criterion preserving existing behaviour is decided by its existing test
  staying green.
- **Ordered so the repo keeps working** after every step; risky or blocking parts come first.
- **Gated at the end**: the final step runs every quality gate the plan's `Quality gates:`
  line names — a plan that ends without the full gate run is unfinished. Its `Covers:` line
  names every criterion the gates re-prove. Where an `AGENTS.md` quality-gate preference
  already runs the full sequence per step, the final step is the recorded proof of the last
  clean run.
- **Tied to the spec**: each step names the acceptance criteria it serves, every criterion is
  covered by at least one step, and pure scaffolding steps say so explicitly. Rule of thumb:
  a step covers one or two criteria — one covering more than three is a split candidate.

If the step list runs long, the spec was probably two specs. Say so before writing the file.

## Plan file structure

Restated here because path-scoped rules load when a matching file is **read**, and a brand-new
plan has not been read yet. `AGENTS.md` and `.claude/rules/plans.md` stay authoritative — if
this ever diverges, follow them and fix this file:

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

Two or three sentences: the strategy, the chosen technologies or patterns with a one-line
why each (long-lived decisions go dated to `systemPatterns.md` — link, don't duplicate),
why the steps are ordered this way, and which `.memory-bank/*` files the implementation
will touch.

**Quality gates:** <the commands from `techContext.md` that Verify lines draw on; the
final step runs them all>

## Research

- Survey: impacted files and callers, implicit contracts, assumptions this plan rests on
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
- **Baseline:** _(commit hash before the first step commit; set when work starts)_
- **Environment:** deviations from `techContext.md` only — standard commands live there
````

## Mode B — resume an existing plan

1. Read the plan first, then the spec. `Current step` and `Session handoff` say where the work
   stands — verify that against `git status --short` and the actual code before trusting it.
   If the pair still sits in `backlog/` or the spec still says `Draft` while work is starting,
   propose the move to `active/` + `In Progress` first — `/sdd-lifecycle` performs it.
2. Report in three lines: what is done, what is next, what blocks it. If
   `activeContext.md`'s `Last updated` is older than the newest commit, say so — the
   dashboard is stale.
3. Continue from the next open step only when the user asks you to. After each finished step,
   update the plan **in the same change set**: tick the box, fill `Notes`, write the real paths
   into the traceability table, move `Current step`, refresh `Last updated` and the handoff.
   Then propose one commit named after the step ID — git writes stay behind the Ask-first gate
   in `AGENTS.md`.
4. When every step is ticked and its criteria hold, set the plan status to `Done` and point the
   user at `/sdd-compile`.

## Mode C — the spec changed

A changed spec whose plan is archived (or still beside it in `done/`, from a pre-1.5 layout)
means an `Implemented` spec is changing: report the impact first (steps 1–3), then propose
the reactivation move — spec back to `active/`, `In Progress` — via `/sdd-lifecycle`; a new
slice of work gets a successor spec instead.

1. Follow the spec's `**Plan:**` line or `## Plan history` to its most recent plan (usually
   in `.specs/plan-archive/`) and read only what the impact needs: which acceptance criteria
   are new, changed, or gone?
2. Read that plan's traceability table **in reverse** — for every touched criterion, list the
   steps and the code paths already built from it. That list *is* the impact: the code a
   change to this requirement reaches. For a criterion that is gone, list its deciding tests
   too — left in place, they keep proving behaviour nobody wants anymore.
3. Report the impact before editing anything: criterion → steps → files → tests, plus what
   becomes obsolete.
4. Then — only after the reactivation move ran and the spec sits in `active/` — write a
   **fresh plan** beside it (structure above, same `NNNN-slug.plan.md` name); a new plan is
   never written beside a `done/` spec. Its Research section links the archived predecessor,
   its steps carry the removals and adaptations from the impact report, and the spec's
   `**Plan:**` line points at it again. The archived plan stays frozen — read it, never
   extend or renumber it. Only a still-active plan (work in flight, never archived) is
   extended in place instead: append steps, **never renumber** existing IDs, strike obsolete
   steps with a one-line reason.

### Impact across several specs

A business change that touches many specs is one impact analysis, not many blind edits:

1. Name every affected spec first — search the criteria, then have the user confirm the list.
2. Revise the specs (`/sdd-specify` revise mode): changed criteria change in place; a
   criterion that no longer applies is struck through with date and reason — IDs are never
   deleted or reused, steps and tests still reference them.
3. For each spec, run steps 1–4 above against its latest plan. No plan anywhere? The AC-IDs
   in test names are the fallback traceability: search the test code for each struck
   criterion's ID to find the tests to remove or retarget.
4. Every removal becomes a plan step with its own `Verify:` line — deleting a test is a
   behaviour change and earns the same evidence discipline as adding one.

## Always

- Write the plan and every report in `DocLanguage`.
- **Planning does not change code, and the plan is not yours to approve.** In Mode A and C, stop
  after writing the file and hand it to the user: name the plan, say how many steps it has, name
  the riskiest one, and say plainly that reading these steps now is cheaper than reading the diff
  later — a wrong step costs hundreds of lines, a wrong line costs one. Ask which steps look
  wrong before anything is implemented. Every other artifact here has a named reader; this one
  is the most expensive to get wrong. Once approved, propose one commit of the plan (Ask-first
  gate) — then stop again. **Approval of the plan approves the document, never the start of
  work**: implementation begins only on the user's explicit start signal — e.g.
  "start T-001", given after the `/sdd-lifecycle` move — and "the plan looks good" is not
  that signal. In Mode C the same explicit go applies.
- Your own todo or task list is scratch state that dies with the session. The plan file is the
  durable one — when the two differ, the file wins and gets corrected.
- Keep the plan lean — it is a working document, not a design essay. Requirements belong in the
  spec, long-lived decisions in `.memory-bank/systemPatterns.md`.
