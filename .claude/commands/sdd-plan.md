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
| A `Baseline` is to change — `**Plan:** _none_`, it never had one | **C**, without a predecessor |

## Mode A — plan from scratch

1. **Read** the spec, `AGENTS.md` (constraints, `architecture:` snapshot, style preferences),
   and `.memory-bank/techContext.md` plus `.memory-bank/systemPatterns.md` — noting the quality gates
   `techContext.md` records (test, build, lint commands). A missing gate is a
   `techContext.md` finding to report, never something to invent.
   Where `systemPatterns.md` holds `fs-knowledge:` blocks, read them by state and leave them that
   way: an observation is a fact about the code, a `decided` rationale with its provenance is a
   constraint you may rely on, and an `unknown` one is **not** — a step that depends on it says so
   in its own words and offers no substitute reason. Two disagreeing sources under `conflict` stay
   two; picking one is the user's call, not a planning shortcut.
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
5. **Decompose into baby steps** and group them into stages (see below).
6. **Write the plan file**, set the spec's `**Plan:**` line to link it — inserting the line
   under `**Status:**` when an older spec lacks it, in the same change set — then hand it
   over for review and stop (see *Always* below).

## Research (do it, do not skip it)

Plan against current facts, not recollection — you are a language model, not a knowledge
base: names, versions, defaults and idioms drift after training. When a web search or fetch
tool is available, **using it is the preferred path** for every fact you would otherwise
assert from memory: a library, framework or API you cannot verify from the repo ·
version-specific behaviour · a protocol, standard or regulation · a naming or architecture
convention under debate · any pattern where your knowledge may be stale. Never quietly
downgrade such a fact to an assumption just to skip the lookup. Web lookups are network
access — the Ask-first gate in `AGENTS.md` applies: name the lookups and get one yes for
the whole research batch. Only when no web tool exists at all, mark the affected steps as
assumptions **and** hand the user ready-made search queries ("googling these for me raises
the plan's quality: …") so the knowledge can still be pulled in. The same applies when the
user declines the research batch: say so plainly and never present a guess as fact.

- If your tool supports subagents, delegate each lookup to an isolated agent and take back
  only the distilled finding plus its source — raw pages crowd out planning judgement.
  Without subagents, do the lookups inline; the recording duty below binds either way.
- Check the version actually used in the repo (lock file, manifest) before trusting a doc page.
- Prefer official documentation, release notes, and the project's own repository.
- Record every source under `## Research`: title, link, one line on what it settled, and the
  date you retrieved it. No link, no claim.

## Baby steps

Restates the step discipline from `.claude/rules/plans.md`, which stays authoritative — a
brand-new plan has not loaded that path-scoped rule yet. A step is one focused change that
can be finished and checked on its own:

- **One behaviour slice per step** — the test and the code for the criteria that the same
  tests decide and the same code serves, typically three to five. Never split by layer or
  file: a stub, a type, an export or a test without its code is not a step — it lands in the
  step that needs it. Every step costs a full red-green cycle and a tick; evidence is
  recorded per criterion, so cutting a slice thinner buys no extra proof.
- **Small enough** to complete in one sitting and to read in one diff — about 150 changed
  lines at most.
- **Verifiable**: every step carries a `Verify:` line — a command whose output decides the step
  (test, build, lint, script). Only where the domain genuinely has no machine check — visual
  layout, wording, a third-party sandbox — write `manual: <what a person looks at>` and say why
  no command can settle it. If you cannot state either, the step is too big or too vague; split it.
- **Recorded**: the step's `Verified:` field stays empty until the `Verify:` line was actually
  run. A tick without a recorded result is a claim, not a verification.
- **Red first**: when a step adds a test for a criterion, the test runs red before the
  implementation lands, and `Verified:` records both runs — what was red, then the green
  result. A test that was never red decides nothing. This binds new behaviour only — a
  criterion preserving existing behaviour is decided by its existing test staying green.
- **Ordered so the repo keeps working** after every step; risky or blocking parts come first.
- **Gated at the end**: the final step runs every quality gate the plan's `Quality gates:`
  line names — a plan that ends without the full gate run is unfinished. Its `Covers:` line
  names every criterion the gates re-prove. Where an `AGENTS.md` quality-gate preference
  already runs the full sequence per step, the final step is the recorded proof of the last
  clean run. The docs sync rides in this same step — README, Memory Bank, and the
  `architecture:` snapshot entries it adds or changes, **named exactly**: approving the plan
  confirms exactly those entries (see `/sdd-architecture-update`).
- **Tied to the spec**: each step names the acceptance criteria it serves, every criterion is
  covered by at least one step. Rule of thumb: a step covering one or two criteria with a
  few lines of code is a merge candidate, one covering more than six a split candidate.

If the step list runs long, the spec was probably two specs. Say so before writing the file.

## Stages

Steps are grouped into **stages**: a stage is a run of steps that need no human decision in
between. Work still stops after every step; the stage end is the checkpoint where plan and
Memory Bank are brought level, and the most a user can ask for in one go besides the whole
plan. List them under `## Stages`, one line each:

- A stage is one coherent slice a person can judge at its end, typically two to four steps.
  It ends before any step that needs an Ask-first action or a real decision. A stop that a
  working agreement asks for — a TDD test review — happens inside a stage and does not end it.
- `→` joins steps that build on each other. `‖` joins steps that are **independent**: no
  shared file — each carries a `Files:` line with its own test file; a file both would
  touch (a manifest, a registration, a shared type) gets its own step before or after
  them — neither needs the other's result, and each `Verify:` runs only its own tests and
  passes without the other step. Independent steps may run in parallel later, by subagents
  or other agents.
- **Look for `‖` in every plan, unasked.** Once the steps are cut, check each pair that
  does not build on the other against those criteria and put the pairs that pass into one
  stage; where the work spans separate modules, shape the slices that way. Close the stage
  list with one line on the result: which stages may run in parallel, or `No parallel
  stage:` and the reason. Mark only what the criteria carry — also when the user asks for
  more: name the criterion that fails and what would make it true (the shared contract as
  its own earlier step). Within one module, and whenever in doubt, write `→`: a wrong `‖`
  costs more than it saves.

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

## Stages

1. T-001 → T-002 — <what this stage delivers>
2. T-003 ‖ T-004 — independent: separate files
3. T-005 — docs sync and final gates

Parallel: stage 2 — or `No parallel stage: <reason>`.

## Steps

### T-001 — <short imperative title>

- **Covers:** AC-001, AC-002
- **Do:** what changes, in which files or modules
- **Verify:** the command whose output decides this step (or `manual: …` plus the reason)
- [ ] **Verified (T-001):** —
- **Notes:** —

### T-003 — <short imperative title>

- **Covers:** AC-003
- **Files:** `src/…`, `tests/…` _(only on `‖` steps: every file the step may touch)_
- **Do:** …
- **Verify:** …
- [ ] **Verified (T-003):** —
- **Notes:** —

## Traceability

| Acceptance criterion | Steps | Code / files | Test | State |
| --- | --- | --- | --- | --- |
| AC-001 | T-001, T-004 | — | — | open |

The checkbox sits on the `Verified` line and carries the step ID, so finishing a step is one
unambiguous edit: tick it and replace `—` with date · command · result. `Notes:` stays `—`
unless the step deviated from `Do:` or found something a later step needs. `Code / files`
and `Test` are filled with real paths and the deciding test at the stage end, not per step.

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
   Ticks are written per step, everything else at the stage end: after an interrupted session
   the ticks and the code lead, so bring traceability and handoff level with them first. A
   return note in `.memory-bank/` (`activeContext.<…>.md`) is parallel work that came back
   and is not integrated yet — integrate it first, per `.claude/rules/parallel-work.md`.
   If the pair still sits in `backlog/` or the spec still says `Draft` while work is starting,
   propose the move to `active/` + `In Progress` first — or, when the user has already given
   the explicit start signal, perform it per *Always* below; `/sdd-lifecycle`'s procedure
   governs either way.
2. Report in three lines: what is done, what is next, what blocks it. If
   `activeContext.md`'s `Last updated` is older than the newest commit, say so — the
   dashboard is stale.
3. Continue from the next open step only when the user asks you to — and then **one step**,
   as `.claude/rules/plans.md` defines it (loaded now that the plan is read; it is
   authoritative): name the step in one line, work it to its end, make the one
   tick-and-`Verified` edit, then stop with a short report and a commit proposal carrying
   the step ID — git writes stay behind the Ask-first gate in `AGENTS.md`. A whole stage or
   the whole plan in one go only when the user asks for it. A stop that a working agreement
   asks for — the test review — holds in every case and is never collected in advance. A
   plan with `‖` stages: ask once how to run them, as the rule describes.
4. When every step is ticked and its criteria hold, set the plan status to `Done` and stop.
   Recommend `/sdd-compile` as the user's next command — in a new session, best with a strong
   model from another model family than the one that implemented. Never start it yourself: a
   "continue" is not that command.

## Mode C — the spec changed

A changed spec whose plan is archived (or still beside it in `done/`, from a pre-1.5 layout)
means an `Implemented` spec is changing: report the impact first (steps 1–3), then propose
the reactivation move — spec back to `active/`, `In Progress` — via `/sdd-lifecycle`; a new
slice of work gets a successor spec instead. A changed `Baseline` is the same case without a
predecessor — it has no plan by design: build the impact report from its `## Technical
Reference` and the survey instead of a traceability table, then reactivate and write its
first plan exactly as below.

1. A `Baseline` has no predecessor: check that it carries `**Clarified:**` — if not, run
   `/sdd-clarify` on it first — then build the impact from its `## Technical Reference` and
   the survey, and go on at step 3. Otherwise follow the spec's `**Plan:**` line or
   `## Plan history` to its most recent plan (usually
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
   extended in place instead: append steps and their stage line, **never renumber** existing IDs,
   strike obsolete
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
  after writing the file and hand it to the user: name the plan, say how many steps and stages it
  has and
  which stages may run in parallel (or that none can), name the riskiest one, and say plainly that
  reading these steps now is cheaper than reading the diff
  later — a wrong step costs hundreds of lines, a wrong line costs one. Ask which steps look
  wrong before anything is implemented. Every other artifact here has a named reader; this one
  is the most expensive to get wrong. Once approved, propose one commit of the plan (Ask-first
  gate) — then stop again. **Approval of the plan approves the document, never the start of
  work**: implementation begins only on the user's explicit start signal — "start T-001",
  "implement" — and "the plan looks good" is not that signal. The start signal also covers
  a still-pending backlog → active move: perform it as part of starting, following
  `/sdd-lifecycle`'s procedure exactly as if the user had typed it (its body lives in
  `.claude/commands/sdd-lifecycle.md`), with the start signal counting as the yes to its
  move proposal — only the commit, and the tracker transition when the spec carries a
  `**Ticket:**` line, stay behind the Ask-first gate — ask for the transition explicitly —
  and say so. In Mode C the same explicit go applies.
- Your own todo or task list is scratch state that dies with the session. The plan file is the
  durable one — when the two differ, the file wins and gets corrected.
- Keep the plan lean — it is a working document, not a design essay. Requirements belong in the
  spec, long-lived decisions in `.memory-bank/systemPatterns.md`.
