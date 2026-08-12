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

Speak plainly, in `DocLanguage`, throughout: assume the user has never seen this
workflow. Every question and every gate opens with one short line saying why it is
asked and what the answer changes, and every term of art (worklist, unit, tier,
scout) gets a one-clause introduction on first use. Minimal and precise — one line
of why, never a lecture.

## Where am I? (state router — the files decide, not the conversation)

- No `.sdd-scan/_worklist.md` → run **Phase A**, then stop at the worklist gate.
- Worklist exists with pending units → run **Phase B** on pending units only.
- All units done, or the user says "distill what we have" → run **Phase C**.

## On a fresh start — say what this buys and costs (once, in `DocLanguage`)

On a fresh start (no worklist yet) this briefing is the **first visible output** —
before any question and before Phase A runs, and equally when `/sdd-setup` routed
here. Three short, plain points (~6 lines total), then proceed:

- **Value:** the fingerprint replaces exploration. Agents in every later session — in both
  tools — jump straight to the right place instead of searching, and technical planning
  (specs, reviews, onboarding) starts from a shared, verified map.
- **Cost:** the scan is token-intensive — scouts read code in isolated contexts. Name a
  planning figure (~50k tokens per analysis unit; large repositories multiply), say that
  cost scales with the models in use and can get expensive, and that a capable model pays
  off for synthesis while the scouts inherit the session model — choosing a cheaper model
  for the scan session, or pinning one locally in the scout agent files, is the biggest
  cost lever.
- **Levers:** the worklist gate is the cost dial — defer units (they stay visible under
  `unmapped:`), set `shallow` tiers for periphery, rescan later with a focus path; the
  scan resumes from its files at any point.

Never repeat this on a resume.

## Phase A — Inventory (git only, no other tooling assumed)

Run and use the results of: `git ls-files` (tree shape, file counts per top-level
folder); `git ls-files '*.sln' '*.csproj' '**/package.json' '**/pom.xml' '**/go.mod'
'**/Cargo.toml' '**/pyproject.toml'` (manifests → stack and project graph); per
candidate source folder `git rev-list --count --since="6 months ago" HEAD -- <folder>`
(change frequency → priority); read CI configs and test scripts found via `git
ls-files`. If the repository's own toolchain can print the project graph with commands
that only read project files — no network, no install or restore (e.g. `dotnet sln
list` plus `dotnet list <project> reference`, or workspace declarations straight from
the manifests) — run them here and record the graph in the inventory; when unsure
whether a command is that harmless, skip it — the git inventory carries alone.
Inventory commands are Phase A's job, exactly as command verification is Phase C's
gated job; scouts never run any. Write `.sdd-scan/_inventory.md` — facts only, no
interpretation. Then
propose analysis units (modules, bounded contexts, projects) with priorities as
`.sdd-scan/_worklist.md`: a table with columns #, Unit, Path, Parent, Depth, Priority,
Tier (deep | standard | shallow), Status (pending | split | done | synthesized), Report.
Propose `deep` when churn is high **or** centrality is high (many incoming references) —
a low-churn shared kernel or contracts folder is a root, not periphery. Propose shared
or contract-like locations (shared kernels, contract folders, cross-service event
definitions, service defaults) as their own high-priority units — each seam is then
read deeply by exactly one scout and only referenced by its neighbours.

Cut coarse-first: ~10–15 units for a mid-size repository is the anchor, and the
operative test is that a unit can carry its own ≤ 120-line report with its own
pattern — a unit whose report would mostly repeat a sibling's patterns, or describe
no behavior at all (assets, generated output, lock artifacts), is not a unit: fold
it in, one inventory line suffices. The asymmetry is why coarse wins: too coarse
self-heals (the split rule fires on evidence — a bursting report budget is the
proof), while too fine has no merge trigger at runtime and only the bill notices.
Where the project graph was extracted, it is the primary cut input: one unit per
weakly coupled subgraph, not one per folder.

## Worklist gate (the only gate this command adds)

Open the gate in plain words: one line on what this list is (the scan's work plan —
which parts of the code get how much attention), one line on why the user decides
(boundaries and depth are judgment calls, and deeper costs more). Then present the
proposed units, priorities and tiers, state the cost of the cut explicitly ("N units
≈ N/3 scout rounds at the default cap"), and offer the plain default ("answer OK to
accept the proposal"). Say what you guessed, ask what to merge, split, defer or
re-tier, apply the answer, mark the worklist confirmed. Never proceed on an
unconfirmed worklist.

## Phase B — Scouts (map)

If your environment can delegate work to an isolated agent with its own context
window — both supported tools can, via the repository's `sdd-scout` agent definition —
dispatch one scout per pending unit, at most three in parallel — a declared default,
not a correctness condition: the worklist makes any degree of parallelism safe; the
cap protects the permission-prompt experience and the tools' concurrency limits, and
an explicit user request ("run up to N scouts") overrides it. If your environment
cannot delegate, do the units yourself, strictly one unit per turn, persisting the
report and updating the worklist before starting the next.

Every delegation restates for the scout (its rule file may not be loaded in a fresh
context; `.claude/rules/architecture-map.md` stays authoritative): write only
`.sdd-scan/reports/<unit-slug>.md`, in `DocLanguage` from `AGENTS.md`; return at most
five summary lines; the report schema — Purpose (2 sentences) · Entry points
(exact paths) · Internal pattern (path patterns, e.g. "Application/<Feature>/ holds
Command + Handler + Validator") · Dependencies in/out (concrete contract or event
files) · Deviations from repo conventions · Traps and frozen zones · Observed,
undocumented decisions (candidates for systemPatterns.md); and the leaf-report
budget — the schema in ≤ 120 lines, where an honest report that cannot fit is the
split rule firing, never a reason to compress harder.

<!-- Authority split, on purpose: procedural rules — the read ladder, the full-read
     caps and the audit header lines — are owned HERE in the delegation block. The
     artifact budgets (≤120-line report, ≤40-line map) and the report schema are owned
     by .claude/rules/architecture-map.md. Do not unify the two homes. -->

Every delegation also carries the working discipline; it binds scouts and the
sequential fallback alike. The read ladder: **locate first** — if your environment
offers a code outline, symbol index, or workspace/semantic search, use it to locate
candidates before reading, treating its results as hints to verify by reading, never
as facts; without such an index, locate via glob and grep (naming patterns,
registrations, contracts). **Head-read is the default**: the first ~60 lines or an
outline — imports, namespace, signatures, doc comment. **Full reads only on
trigger**, of two kinds: mandatory (entry points, the composition root,
contracts/events, and one representative per recognized pattern) and uncertainty
(the head contradicts the hypothesis, name and content disagree, trap suspicion).
A name is a hypothesis; the head is the test — never state a role derived from a
name or path without confirming it in content, and record every name↔content
mismatch as a trap finding: it is high-value fingerprint material. Stop reading when
a new file no longer changes the report.

Full reads are budgeted: the delegation names the unit's tier and assigns the cap
(standard 10 · seam/deep up to 20 · shallow ≤ 3), and the report header carries
`tier:` plus the audit line `full reads: n/cap`. Overflow against an assigned budget
is the split rule firing — never silence, never compression. A shallow unit still
reads heads, never names alone: minimum is the manifest plus the heads of the three
to five most-referenced files.

Read and search only; run no commands and build nothing — the scan stays
reproducible and side-effect-free, and command verification is Phase C's gated job.
Beyond the unit's own paths, record the touchpoint file as a dependency and stop —
the neighbour's inside belongs to the neighbour's scout. Include the siblings'
worklist rows (unit, path, one-line purpose), so the scout knows where its unit
ends — and, when Phase A extracted a project graph, the unit's own graph digest:
its in/out edges plus the top-level shape, never the full graph, passed as located
hints to verify by reading, never as facts. A runtime coupling the static graph
does not show is a trap finding — the graph↔runtime sibling of the name↔content
rule.

**Split rule:** a unit that is not coherent and graspable (heuristics: no single
recognizable pattern, clearly beyond ~150 relevant files, or an honest report that
will not fit the ≤ 120-line budget) gets no shallow report —
the scout returns a structural note plus proposed child units instead; add them to
the worklist with Parent and Depth, mark the unit `split`. Depth is capped at 3;
beyond that, ask the user. After every scout return, update the worklist **in the
same change set** — a report whose worklist row did not move is not done, and a
report missing its `tier:` and `full reads: n/cap` header lines is equally not
done: complete the header from the scout's return (or re-ask) before marking the
row.

## Phase C — Synthesis (reduce) and distillation

Bottom-up: when all children of a node are done, merge their reports into one report
for that node — keep only what is decision-relevant at that level, replace child
details with the pattern they share — until the root report is the big picture.
Where two reports describe the same seam differently, the contradiction is a
synthesis finding to resolve here — not a scout error.

Curation MUSTs for everything that survives: nothing an agent can infer from the code
in seconds; path patterns over path lists; every documented command was executed once
(commands that reach the network, such as package restores, fall under the ask-first
rule in `AGENTS.md` — ask, or mark the command unverified); the *why* behind observed
decisions goes, dated and source-linked, to `.memory-bank/systemPatterns.md`; stack,
build, run and test facts go to `.memory-bank/techContext.md`. Everything curated —
snapshot wording, module maps, Memory Bank entries — is written in `DocLanguage` from
`AGENTS.md`, and that includes the snapshot's own values, not just prose around them.
The snapshot obeys the same curation bar: every `entrypoints:`/`modules:` line carries
its path plus a one-clause purpose or trap — never a bare path. A snapshot that a
directory listing could have produced is not a fingerprint.

**Budget cascade:** target the `architecture:` snapshot alone. Measure against the
cap in `.claude/rules/constitution.md`. Only when eviction would hit navigation
facts, move per-module detail to `.architecture/<unit>.md` (≤ 40 lines each — this
restates `.claude/rules/architecture-map.md`, which is authoritative) and keep in the
snapshot one line and a `map:` reference per module, plus `coverage:` — counting
shallow-tier units separately, e.g. "9/14 mapped (2 shallow)" — and an `unmapped:`
list instructing agents to explore carefully there and propose a worklist entry.
Shallow-tier units carry the matching instruction: their facts are head-derived —
verify before relying on them, and on first real contact propose a deepening
worklist entry.

## Self-test before hand-off

The self-test is **mandatory** — the hand-off below stays blocked until its score
and the surface-cue probe's outcome are recorded for the delta report. A distilled
result that cannot answer navigation questions is not ready to become the snapshot.

Compose ten navigation questions ("Where do you add/change X?") across the mapped
**deep and standard** units — shallow units are excluded from the score. Answer each
in writing using **only** your distilled result — no file access — then verify each
against the repository and score. Fix the fingerprint for every miss you can, and
carry the final score into the delta report.

Then run the surface-cue probe: while composing the questions, mark which snapshot
claims would be plausible from names alone, pick one deliberately — prefer a shallow
unit — and verify it against file contents. A mismatch is a hard fail independent of
the score: correct the fingerprint, then repeat the probe. A name-derived error in
the fingerprint is multiplicative poison — every later session trusts it with the
authority this document exists to carry.

## Hand-off, then cleanup

Never write the snapshot here. Run the `/sdd-architecture-update` workflow exactly as
if the user had typed it (its body lives in
`.claude/commands/sdd-architecture-update.md`), with your distilled findings as the
observed state; its confirmation gate remains the snapshot's only write gate. After a
confirmed merge, set `last deep scan` in the snapshot comment via that same update,
then ask once: "Delete the raw analysis in `.sdd-scan/`? (recommended)".
