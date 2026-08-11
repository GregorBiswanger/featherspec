---
description: Deep architecture scan — recursively analyze an existing codebase into the fingerprint. Resumable; safe to re-run anytime.
argument-hint: "[focus path] (optional)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-architecture-scan workflow. Claude Code runs this file
     directly; GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-architecture-scan.prompt.md. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither. -->

# /sdd-architecture-scan — Deep Architecture Scan

The user may name a focus path after the command; then every phase below applies to
that subtree only, and the final merge touches only its entries.

## Where am I? (state router — the files decide, not the conversation)

- No `.sdd-scan/_worklist.md` → run **Phase A**, then stop at the worklist gate.
- Worklist exists with pending units → run **Phase B** on pending units only.
- All units done, or the user says "distill what we have" → run **Phase C**.

## Phase A — Inventory (git only, no other tooling assumed)

Run and use the results of: `git ls-files` (tree shape, file counts per top-level
folder); `git ls-files '*.sln' '*.csproj' '**/package.json' '**/pom.xml' '**/go.mod'
'**/Cargo.toml' '**/pyproject.toml'` (manifests → stack and project graph); per
candidate source folder `git rev-list --count --since="6 months ago" HEAD -- <folder>`
(change frequency → priority); read CI configs and test scripts found via `git
ls-files`. Write `.sdd-scan/_inventory.md` — facts only, no interpretation. Then
propose analysis units (modules, bounded contexts, projects) with priorities as
`.sdd-scan/_worklist.md`: a table with columns #, Unit, Path, Parent, Depth, Priority,
Status (pending | split | done | synthesized), Report.

## Worklist gate (the only gate this command adds)

Present the proposed units and priorities. Unit boundaries need human interpretation:
say what you guessed, ask what to merge, split or defer, apply the answer, mark the
worklist confirmed. Never proceed on an unconfirmed worklist.

## Phase B — Scouts (map)

If your environment can delegate work to an isolated agent with its own context
window — both supported tools can, via the repository's `sdd-scout` agent definition —
dispatch one scout per pending unit, at most three in parallel. If it cannot, do the
units yourself, strictly one unit per turn, persisting the report and updating the
worklist before starting the next.

Every delegation restates for the scout (its rule file may not be loaded in a fresh
context; `.claude/rules/architecture-map.md` stays authoritative): write only
`.sdd-scan/reports/<unit-slug>.md`, in `DocLanguage` from `AGENTS.md`; return at most
five summary lines; and the report schema — Purpose (2 sentences) · Entry points
(exact paths) · Internal pattern (path patterns, e.g. "Application/<Feature>/ holds
Command + Handler + Validator") · Dependencies in/out (concrete contract or event
files) · Deviations from repo conventions · Traps and frozen zones · Observed,
undocumented decisions (candidates for systemPatterns.md).

**Split rule:** a unit that is not coherent and graspable (heuristic: no single
recognizable pattern, or clearly beyond ~150 relevant files) gets no shallow report —
the scout returns a structural note plus proposed child units instead; add them to
the worklist with Parent and Depth, mark the unit `split`. Depth is capped at 3;
beyond that, ask the user. After every scout return, update the worklist **in the
same change set** — a report whose worklist row did not move is not done.

## Phase C — Synthesis (reduce) and distillation

Bottom-up: when all children of a node are done, merge their reports into one report
for that node — keep only what is decision-relevant at that level, replace child
details with the pattern they share — until the root report is the big picture.

Curation MUSTs for everything that survives: nothing an agent can infer from the code
in seconds; path patterns over path lists; every documented command was executed once
(commands that reach the network, such as package restores, fall under the ask-first
rule in `AGENTS.md` — ask, or mark the command unverified); the *why* behind observed
decisions goes, dated and source-linked, to `.memory-bank/systemPatterns.md`; stack,
build, run and test facts go to `.memory-bank/techContext.md`. Everything curated —
snapshot wording, module maps, Memory Bank entries — is written in `DocLanguage` from
`AGENTS.md`.

**Budget cascade:** target the `architecture:` snapshot alone. Measure against the
cap in `.claude/rules/constitution.md`. Only when eviction would hit navigation
facts, move per-module detail to `.architecture/<unit>.md` (≤ 40 lines each — this
restates `.claude/rules/architecture-map.md`, which is authoritative) and keep in the
snapshot one line and a `map:` reference per module, plus `coverage:` and an
`unmapped:` list instructing agents to explore carefully there and propose a worklist
entry.

## Self-test before hand-off

Compose ten navigation questions ("Where do you add/change X?") across the mapped
units. Answer each in writing using **only** your distilled result — no file access —
then verify each against the repository and score. Fix the fingerprint for every
miss you can, and carry the final score into the delta report.

## Hand-off, then cleanup

Never write the snapshot here. Run the `/sdd-architecture-update` workflow exactly as
if the user had typed it (its body lives in
`.claude/commands/sdd-architecture-update.md`), with your distilled findings as the
observed state; its confirmation gate remains the snapshot's only write gate. After a
confirmed merge, set `last deep scan` in the snapshot comment via that same update,
then ask once: "Delete the raw analysis in `.sdd-scan/`? (recommended)".
