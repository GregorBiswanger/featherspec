---
description: Reverse specification — reconstruct existing behaviour from code, tests or a plan into a human-validated Baseline spec. Resumable.
argument-hint: "[target: path | folder | symbol | plan file] (optional)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-reverse-specify workflow, for Claude Code and GitHub Copilot
     alike: Claude Code runs this file directly, GitHub Copilot reaches it through the thin
     loader in .github/prompts/sdd-reverse-specify.prompt.md. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither. -->

# /sdd-reverse-specify — Reverse Specification

The reverse counterpart of `/sdd-specify`. `/sdd-specify` asks *what should the system do?*
This command asks *what does the system already appear to do — and which of it is valid
business behaviour?* Two truths govern every step: code shows what the system does, never
what was meant; and nothing reconstructed becomes a specification without a human's yes.

The user may name a target after the command — a file, folder, symbol, module, API or a
technical plan. The target is the **analysis boundary, never the spec boundary**: one class
may carry several capabilities, and one capability may span several classes.

Speak plainly, in `DocLanguage`, throughout: assume the user has never seen this workflow.
Every question and every gate opens with one short line saying why it is asked and what the
answer changes; every term of art (capability, candidate, marker, scout) gets a one-clause
introduction on first use. One question per message. Never a code fence around a question.

## Where am I? (state router — the files decide, not the conversation)

- No `.sdd-reverse/_worklist.md` → **Phase 0**, then **Phase A**; stop at the map gate.
- A named target the worklist does not cover yet → Phase A on that target only; add its
  rows, keep the rest.
- A row `validating` → offer to continue its validation (**Phase C**) or to leave it for
  later; "later" is a full answer and moves on.
- A row `pending` → **Phase B** for the highest-priority pending row — one capability per
  run unless the user asks for more — then synthesis, then Phase C.
- A candidate the user calls sufficiently validated → **Phase D**.
- No row `pending` or `validating` → the cleanup question, nothing else.

Row status vocabulary: `pending | analyzed | validating | promoted | deferred | rejected`.

## On a fresh start — say what this buys and costs (once, in `DocLanguage`)

Three short points (~6 lines), before the first question, never on a resume:

- **Value:** behaviour that today lives only in code becomes a spec the team has confirmed —
  later changes start from known rules instead of rediscovery.
- **Cost:** reading agents (scouts) open code in isolated contexts; every capability is one
  analysis round, and rounds cost time and tokens. Nothing is analysed before the map gate.
- **Levers:** reconstruct what is about to change or must be protected; everything else
  stays `deferred` and costs nothing. The work resumes from its files at any point, and
  validation can happen later, by anyone who knows the behaviour.

## Phase 0 — Scope and context

- **With a target:** focused mode. A path is taken as given; a symbol is located by search
  and confirmed in one line; a plan file switches to plan mode — `Origin: Reverse —
  Technical Plan`: the plan is evidence of the intended approach, the implementation is
  evidence of what was delivered, tests and contracts are further evidence. Where they
  disagree, that is a conflict to show — never pick one. The plan stays historical
  evidence; a retrospective plan is never written.
- **Without a target:** ask exactly one question — the whole application, or a specific
  area? The whole application is navigated through the `architecture:` snapshot in
  `AGENTS.md` and the `.architecture/` maps; if the snapshot is still `TBD`, say so and
  recommend `/sdd-architecture-scan` first or a focused run. Never rescan the repository
  blindly here — the fingerprint answers *where*, this command answers *what*.
- **Functional description:** reuse `.memory-bank/projectbrief.md`, existing specs and the
  conversation (including answers `/sdd-setup` collected) and name each source. Ask only
  what is missing, at most four questions: what the software does · who uses it · the
  known main workflows or domains · areas to include or exclude. In focused mode one
  question usually suffices: what is this area for?

Write `.sdd-reverse/_context.md` (≤ 40 lines, `DocLanguage`): mode, target, origin, the
description with a source per statement, inclusions and exclusions. Its first line says it
is a hypothesis: the description and the code are two sources of evidence that may disagree.

## Phase A — Capability map (cheap: no deep reading yet)

Inputs: the snapshot and maps, `_context.md`, and the **heads** of entry points only —
routes, handlers, menus, jobs, the public surface of the target. Locate first (outline,
index, glob, grep), then read heads (~60 lines). More than ~25 head reads means the area is
too big for one map: map module by module along the snapshot and ask which module first.

A **capability** is something an actor can achieve, named verb + object in the actor's
words — "Approve an order". Never a class, controller, table, endpoint or folder name: a
name carrying one is rewritten before anyone sees it. Aim at user-goal level; fold create,
read, change and delete of one thing into one capability unless they differ in actor or
rules. Technical modules are never capabilities; record per row which entry points and
modules the capability spans.

Write `.sdd-reverse/_worklist.md`: a table with columns #, Capability, Actor, Entry
points, Spans, Priority, Status, Candidate. Priority: what the user called about to change
or to be protected first, then the snapshot's central or high-churn modules.

**Map gate (the first of two gates).** One line on what the list is (a guess at what the
software lets people do), one on why the user decides (only people know what counts as one
capability). Show the rows, say what you guessed, ask what to rename, merge, split, defer
or reject, and state the cost in plain terms: "N capabilities = N analysis rounds;
reconstruct first what changes soon — deferring costs nothing". Offer the plain default
("answer OK to accept"). In focused mode the map is small, and the question is which of
the capabilities found in the target to reconstruct. Never proceed on an unconfirmed map.

## Phase B — Evidence (scouts)

If your environment can delegate to an isolated agent with its own context window — both
supported tools can, via the repository's `sdd-reverse-scout` agent definition — dispatch
scouts for **one** capability. A small capability (≈ 15 relevant files or fewer) gets one
scout carrying all three lenses; a larger one gets one scout per lens, at most three in
parallel. Lenses cut by kind of evidence, not by folder, so their reports triangulate:

1. **Entry and flow** — who triggers it, inputs, outputs, authorization, the sequence.
2. **Rules and states** — conditions, thresholds, invariants, transitions, failure paths.
3. **Tests and contracts** — what tests assert, API contracts, schemas; the plan in plan mode.

If your environment cannot delegate, work the lenses yourself, strictly one lens per
turn, persisting the report before starting the next. You stay the orchestrator: you read
`_context.md`, the worklist, scout summaries and the reports of the capability in flight —
never large parts of the repository.

Every delegation restates for the scout (its context is fresh): the capability, actor,
entry points and spans · the lens · the report path
`.sdd-reverse/reports/<capability-slug>.<lens>.md` (lens slug: `flow` | `rules` | `tests` |
`all`) · `DocLanguage` · the schema and the
discipline below · return at most five summary lines.

**Report schema (≤ 120 lines):** header lines `capability:`, `lens:`, `full reads: n/cap` ·
*Observed behaviours* — one line each: the behaviour in domain language · kind (flow | rule
| state | authz | failure | data) · evidence `path:line` · *Literal values* — every
threshold, limit, default, rounding direction and message as written in the code, with
evidence; where the code checks no upper or lower bound, say so · *Inputs and
outputs* in business meaning, no field dumps · *Disagreements* — code vs test, code vs
comment, code vs plan, both pointers, neither chosen · *Unreferenced code* in scope
(`path › symbol`, how checked) · *Open questions*.

**Discipline (binds scouts and the sequential fallback alike):**

- Read and search only; run no commands, build nothing.
- Locate first, head-read by default; full reads where behaviour lives — rules live in
  bodies. Budget: 15 full reads for one lens, 20 for all lenses. Overflow is the split
  rule firing: return a structural note proposing sub-capabilities, never a thinner report.
- A name is a hypothesis. Never state a behaviour from a name, a comment or a test title
  alone — confirm it in a body and cite the line.
- A value is copied, never recalled: what systems of this kind usually do is not evidence.
- Say what the code does. Never why, never "intended", never "should".
- Before reporting a behaviour, check its code is reachable from an entry point (search for
  callers). Unreachable code goes under *Unreferenced code*, never under *Observed*.
- The target bounds where you start, not where the capability ends: follow its flow to the
  touchpoints it needs, record them, stop there.

After every scout return, set the worklist row to `analyzed` in the same change set.

## Synthesis — the candidate

Read only this capability's reports. **Evidence probe first:** open three cited pointers —
prefer literal values and anything plausible from names alone. A pointer that does not
resolve, or does not support its statement, is a hard fail: fix or drop the statement,
then probe again. A name-derived rule in a spec is inherited by every later change.

Write `.sdd-reverse/candidates/<capability-slug>.reverse-spec.md`, in `DocLanguage`:

```markdown
# <Capability>

> Reconstructed from technical evidence — not validated yet. Not a requirement.
> Continue with /sdd-reverse-specify.

**Status:** Reverse candidate
**Origin:** Reverse — Brownfield
**Analyzed at:** <short commit hash, or "no commit"> · <date>
```

The body uses the document structure and the five criterion shapes of `/sdd-specify`
(`.claude/commands/sdd-specify.md` stays authoritative — this is a restatement): Summary ·
Users and roles · Scope (in scope / out of scope: boundaries the code visibly draws) ·
Functional flow · Business rules (`BR-NN`) · Error cases · Acceptance criteria (`AC-NNN`,
**shall**, describing what *is*) · Assumptions · Open points — then `## Conflicts` and
`## Evidence`. Promotion is then a move, not a rewrite.

**State each behaviour once.** *Acceptance criteria* carry the testable behaviour;
*Business rules* hold only what several criteria share — an invariant, a policy, a literal
limit — and never repeat a criterion in other words. A small capability may need three
criteria and no rule: length is not thoroughness, and every line is a line someone must
validate. Implementation qualities — a secret in the source, a missing index, a slow path
— are not behaviour an actor can observe: at most one line each under *Open points*.

**Markers.** Every business rule and every criterion carries exactly one, right after
its ID. They are the whole vocabulary — never a percentage, a score or a probability:

- `[Inferred]` — read in a reachable code body, unambiguous, and no source disagrees. A
  second agreeing source (test, contract, plan) is noted in *Evidence*.
- `[Uncertain]` — there is evidence, but its functional meaning is unclear: sources
  disagree, the behaviour looks accidental (dead branch, workaround, leftover), or only
  names and comments hint at it. Say in half a sentence what makes it uncertain.
- `[Confirmed]` — a human validated it. Synthesis never writes this marker; only Phase C does.

**Language lint before writing.** A rule or criterion that names a class, method, file,
table, endpoint, HTTP verb or status code is rewritten to the behaviour an actor can
observe; the technology belongs in `## Evidence` (`| ID | Evidence | Second source |`,
pointers only — never pasted code). One worked example:

```text
Bad:  AC-004 [Inferred] OrderController.Approve returns 403 when user.Role != "Manager".
Good: AC-004 [Inferred] If someone other than a manager tries to approve an order, then
      the system shall refuse the approval.      (Evidence: src/Orders/Approval.cs:41)
```

**Never settle what only people can settle.**

- Sources disagree → one row in `## Conflicts` (`C-NN | sources | what each says, with
  pointers`), the affected rule `[Uncertain]`. A conflict needs two sources that cannot
  both be true — description vs code counts; a source that is merely silent or less
  precise does not.
- Rules and criteria are written from what the code does. A test shows what someone
  expected: a test the code cannot satisfy is a conflict, never the source of an
  `[Inferred]` statement.
- Behaviour that looks like a bug, a workaround or legacy is written `[Uncertain]` with
  the reason — never dropped, never stated as a rule.
- Unreferenced code never becomes a rule or a criterion; it appears once under *Open
  points* as a question.
- More than 25 criteria is a split proposal at the next gate, not a longer file.

Set the row to `validating`, name the candidate path in its last column. Keep the session
dashboard honest per `AGENTS.md`: one or two lines in `.memory-bank/activeContext.md` naming
this command and the worklist — never candidate content, and nothing in `AGENTS.md` ever
points into `.sdd-reverse/`.

## Phase C — Validation (the second gate, step by step)

Open with three lines: what was reconstructed (counts per marker and conflicts), that the
user decides what is valid business behaviour, and that anyone who knows the behaviour may
answer — a product owner for business rules, a developer for technical-domain rules. Two
answers are always open: **unsure** settles one question as "don't know", and **later**
parks the whole validation at any point — the row stays `validating`, and the candidate
file keeps the state between sessions and between people.

Order: conflicts first, then `[Uncertain]`, each as one question per message; then
`[Inferred]` in rounds of five to seven as **one** numbered confirmation — "confirm all, or
name the numbers that are wrong or unsure". A question states the behaviour in plain
words, where the evidence sits in one clause, and asks whether it is intended business
behaviour. The user may **confirm**, **correct**, **reject** or stay **unsure**; for a
conflict or a suspicious behaviour also offer what it might be: intended · bug · legacy ·
workaround · undocumented exception · unknown.

Apply every answer to the candidate at once: confirmed → `[Confirmed]`; corrected → the
corrected wording, `[Confirmed]`; rejected → removed, one line under *Open points*
("rejected as not a business rule: …"); unsure → stays `[Uncertain]`, tagged `reviewed`.
When the valid behaviour differs from what the code does — a bug verdict, or a correction
the code contradicts — ask once to be sure, then write the valid behaviour as the rule and
record under *Open points*: `Known deviation: the implementation does X (pointer) —
declared <verdict> by <who>`. This command never changes code; a fix is normal future work
through `/sdd-specify`. A reason the validator volunteers is kept with the rule,
attributed; never ask for reasons unprompted.

Ask once under which name the validation is recorded, offering `git config user.name` as
the default; several validators are listed; no meaningful name → no line. Close with the
question whether the user considers this capability sufficiently validated — a no leaves
the candidate where it is.

## Phase D — Promotion to a normal Baseline spec

Precondition: no unreviewed `[Inferred]` or `[Uncertain]` remains — validate it now, or
the user moves it to *Open points* as reviewed and unresolved. Then propose all writes in
**one** confirmation and act only on a yes:

- Write `.specs/done/NNNN-<slug>.md` (next free number across `.specs/`). Header, restating
  `.claude/rules/specs.md`: `**Status:** Baseline` · `**Plan:** _none_` · `**Origin:**
  Reverse — Brownfield` (or `Reverse — Technical Plan`) · `**Validated by:** <names> ·
  <date>`. `Origin` is provenance only — from here on it is a normal spec.
- Markers disappear when everything is confirmed. Reviewed, unresolved uncertainty stays
  visible under *Open points*, each line keeping `[Uncertain]`.
- Replace `## Evidence` and `## Conflicts` by `## Technical Reference`, **≤ 8 lines**, paths
  and symbols only: architecture reference (`AGENTS.md` → module) · primary implementation
  (a path pattern) · main contract · deciding tests · in plan mode the historical plan ·
  `analysed at <hash> · <date>`. No directory trees, no code walkthroughs, no invented
  design history. The spec must read complete without the reports.
- **Never write a plan**, never touch `backlog/` or `active/`: a Baseline lives in `done/`
  without one. A later change runs the normal flow — `/sdd-plan` reads the Technical Reference.
- Delete the candidate and this capability's reports (this workflow's own files), set the
  row to `promoted` with the spec path, add one line to `.memory-bank/activeContext.md`.

Hand off: name the file, what stayed open, and recommend `/sdd-clarify` on it in a fresh
context — a Baseline needs that pass. Propose the commit (Ask-first, per `AGENTS.md`).

## Cleanup

When no row is `pending` or `validating`, ask once, in `DocLanguage`: "Delete the raw
analysis in `.sdd-reverse/reports/`? (recommended)" — and, when no `deferred` row remains,
the whole `.sdd-reverse/` folder. Deferred rows keep `_context.md` and `_worklist.md`
alive: they are the memory of what was seen and not yet reconstructed.
