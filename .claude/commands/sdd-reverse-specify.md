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

## Your role and the one outcome

You work as a requirements engineer documenting an existing system for its product owner.
The outcome is one Baseline spec per capability: what the system does **today**, in business
language, confirmed by a person who knows it. The code shows what the system does, never
what anyone meant. People confirm that you understood the system; how it *should* behave is
never asked here — that is decided later, through `/sdd-specify`, starting from the
Baseline. Why it matters: the person answering may be a trainee who cannot tell when a
question drifts, and every later change is planned against what they sign — a wish stated
as fact, or a surprise that got lost, misleads everyone after them.

Done means: every rule and criterion describes behaviour the code has, and no belief or wish
changed one; every behaviour that could harm or mislead a person is recorded as a
*finding*; nothing became a spec without a person's yes; and no question asked for a value,
limit, message or decision the system does not already have. Questions about the work
itself — which capabilities, whose name, promote now? — stay the user's to answer.

The user may name a target after the command — a file, folder, symbol, module, API or a
technical plan. The target is the **analysis boundary, never the spec boundary**: one class
may carry several capabilities, and one capability may span several classes.

Speak plainly, in `DocLanguage`, throughout: assume the user has never seen this workflow.
Every gate opens with one short line saying why it is asked and what the answer changes;
every term of art (capability, candidate, marker, scout, finding) gets a one-clause
introduction on first use. One question per message. Never a code fence around a question.

## What belongs in a Baseline — sort every observation into exactly one place

Spec content is whatever an actor would notice if it were different: what they can do and
see, a limit, a message, how long they wait, what happens to them when something fails —
how many items fit in a basket, what a declined payment does to an order. How the system
achieves it is not spec content. Here all of it is written as the system behaves **now**:

- **Rule or criterion** — behaviour the code has, as an actor experiences it.
- **Finding** — behaviour that could harm or mislead a person, or a missing check that lets
  that happen: someone gets what is not theirs, loses what they entered, pays twice, is left
  without an answer. Stated as what the person experiences, never as a guess at why, and a
  finding whatever anyone replies. Torn between the two: harm → finding, otherwise rule.
- **Evidence only** — how it is done: storage, libraries, retries, classes. Before filing
  something here, ask what a person experiences because of it — state that survives a
  failed step and is used again later is the classic hidden finding.
- **Not here** — what should be: a value to choose, a limit to add, a decision to take.
  That is the question of `/sdd-specify`, once someone decides to change the system.

```text
Rule:      BR-02 [Inferred] A member may borrow at most five books at the same time.
Finding:   A member whose card has expired can still reserve a book; the reservation is
           never handed out.          (not: "the pickup job skips expired cards")
Finding:   A member can reserve any number of books — one member can block every copy.
Evidence:  The loan list is cached in the browser.   → a pointer in ## Evidence, no text
Not here:  How many days should the grace period be? → never asked
```

## Where am I? (state router — the files decide, not the conversation)

- No `.sdd-reverse/_worklist.md` → **Phase 0**, then **Phase A**; stop at the map gate.
- A named target the worklist does not cover yet → Phase A on that target only; add its
  rows, keep the rest.
- A row `validating` → offer to continue its validation (**Phase C**) or to leave it paused;
  paused, go on with the next line.
- A row `analyzed` → the lenses still missing (**Phase B**), then synthesis, then Phase C.
- A row `pending` → **Phase B** for the highest-priority pending row — one capability per
  run unless the user asks for more — then synthesis, then Phase C.
- A candidate the user calls sufficiently validated → **Phase D**.
- No row `pending`, `analyzed` or `validating` → the cleanup question, nothing else.

Row status vocabulary: `pending | analyzed | validating | promoted | deferred | rejected`.

## On a fresh start — say what this buys and costs (once, in `DocLanguage`)

Three short points (~6 lines), before the first question, never on a resume:

- **Value:** behaviour that today lives only in code becomes a spec the team has confirmed —
  later changes start from known behaviour instead of rediscovery.
- **Cost:** reading agents (scouts) open code in isolated contexts; every capability is one
  analysis round, and rounds cost time and tokens. Nothing is analysed before the map gate.
- **Levers:** reconstruct what is about to change or must be protected; everything else
  stays `deferred` and costs nothing. The work resumes from its files at any point, and
  anyone who knows how the system behaves can confirm it, in a later session.

## Phase 0 — Scope and context

- **With a target:** focused mode. A path is taken as given; a symbol is located by search
  and confirmed in one line; a plan file switches to plan mode — `Origin: Reverse —
  Technical Plan`: the plan is evidence of the intended approach, the implementation is
  evidence of what was delivered, tests and contracts are further evidence. Where they
  disagree, the statement says what was delivered, and the plan's version stands beside it
  as a conflict. The plan stays historical evidence; a retrospective plan is never written.
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
points, Spans, Priority, Status, Candidate — column names and status values stay in English,
commands read them; cells are in `DocLanguage`. Priority: what the user called about to change
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
outputs* in business meaning, no field dumps · *Leftover state* — what stays stored after a
failed or cancelled step, and where it is used again · *Disagreements* — code vs test, code vs
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

Once every lens of the capability has its report, set the worklist row to `analyzed` in the
same change set.

## Synthesis — the candidate

Read only this capability's reports and `_context.md`. **Evidence probe first:** open three
cited pointers — prefer literal values and anything plausible from names alone — and the
pointer behind every finding. A pointer that does not
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
**shall**, describing what the system does today) · Assumptions (conditions outside the
system, never a belief) · Open points — each line opens with one label in `DocLanguage`:
*Finding* (closing with its pointer) · *Check* (for a developer) · *Wish* · *Removed*; only
reviewed uncertainty keeps its marker instead — then `## Conflicts` and `## Evidence`.
Headings are written in `DocLanguage`, except `## Conflicts`, `## Evidence` and
`## Technical Reference`: commands find those by name. Promotion is then a move, not a
rewrite.

**State each behaviour once.** *Acceptance criteria* carry the testable behaviour;
*Business rules* hold only what several criteria share — an invariant, a policy, a literal
limit — and never repeat a criterion in other words. Summary, flow, scope and error cases
add nothing new: each behaviour they mention also stands, by ID, as a rule or criterion. A
small capability may need three criteria and no rule: length is not thoroughness, and every
line is a line someone must confirm. Implementation qualities — a secret in the source, a
missing index, a slow path — are not behaviour an actor can observe: at most one *Check*
line each.

**Markers.** Every business rule and every criterion carries exactly one, right after
its ID. They are the whole vocabulary — never a percentage, a score or a probability:

- `[Inferred]` — read in a reachable code body, unambiguous, and no source disagrees. A
  second agreeing source (test, contract, plan) is noted in *Evidence*.
- `[Uncertain]` — the code alone does not settle what an actor experiences: sources
  disagree, the outcome depends on something outside the repository (a setting, another
  system), or only names and comments hint at it. After the marker, say in half a sentence
  what makes it uncertain — the brackets hold the marker word alone, never the reason.
  Behaviour that merely looks wrong and a check the code does not make are findings, not
  `[Uncertain]`.
- `[Confirmed]` — a person confirmed it. Synthesis never writes this marker; only Phase C
  does. Findings carry no marker and no ID.

**Language lint before writing.** A rule, criterion or finding that names a class, method,
file, table, endpoint, HTTP verb or status code — or a mechanism: where state is stored, a
library, a build variant, a retry count, a fixed wait — is rewritten to what an actor can
observe; a number stays only when an actor would notice a different value. The technology
belongs in `## Evidence` (`| ID | Evidence | Second source |`, pointers only, each with its
full path from the repository root — never pasted code). Two worked examples:

```text
Bad:  AC-004 [Inferred] OrderController.Approve returns 403 when user.Role != "Manager".
Good: AC-004 [Inferred] If someone other than a manager tries to approve an order, then
      the system shall refuse the approval.      (Evidence: src/Orders/Approval.cs:41)
Bad:  AC-012 [Inferred] The token sits in localStorage; a 30-minute setTimeout clears it.
Good: AC-012 [Inferred] If a signed-in person does nothing for 30 minutes, then the system
      shall sign them out.                       (Evidence: src/auth/session.ts:22)
```

Text an actor sees — a message, a label, a button — is quoted in backticks exactly as the
code has it, character for character: a later test asserts that string. Write `DocLanguage`
in its own script; only a file slug is ever transliterated.

**Keep what the code does apart from what anyone meant.**

- Sources disagree → one row in `## Conflicts` (`C-NN | sources | what each says, with
  pointers`); the statement says what the reachable code does, is `[Uncertain]` and names
  the other source ("a test expects …"). A conflict needs two sources that cannot both be
  true — description vs code counts; a source that is merely silent or less precise does not.
- A test shows what someone expected: a test the code cannot satisfy is a conflict, never
  the source of an `[Inferred]` statement.
- Unreferenced code never becomes a rule, a criterion or a finding; it gets one *Check* line.
- More than 25 criteria is a split proposal at the next gate, not a longer file.

Set the row to `validating`, name the candidate path in its last column. Keep the session
dashboard honest per `AGENTS.md`, and quiet: touch `.memory-bank/activeContext.md` only when
a row changes status — one or two lines naming this command and the worklist, never
candidate content, never after single answers. Nothing in `AGENTS.md` ever points into
`.sdd-reverse/`.

## Phase C — Validation: check your understanding, not their intent

The person confirms that you understood the system — nothing more. Every question is about
what the system does today; what it should do is the question of `/sdd-specify` and is not
asked here, not even as a follow-up. Read the candidate and, to check an answer, the code
the answer is about.

Open with a short paragraph in `DocLanguage`: what was reconstructed (rules and criteria per
marker, findings); that you now check your understanding, and nothing about how the system
should behave is decided here; and the four answers, as words in `DocLanguage` —
**correct** · **not correct** · **don't know** · **pause** (German: stimmt · stimmt nicht ·
weiß nicht · Pause). Any other reply to a statement counts as *don't know*; only the pause
word stops the validation — the row stays `validating`, and the candidate keeps the state
between sessions and between people. Anyone who knows how the system behaves may answer: a
user, a product owner, a developer for technical-domain rules.

Order — a round or a list is one message:

1. `[Uncertain]` statements, conflicts first — one per message.
2. `[Inferred]` statements in rounds of five to seven, as **one** numbered message: "all
   correct — or name the numbers that are not correct or you don't know".
3. All findings in one message, for information — recorded so they are not lost, nothing to
   decide — ending with the question under which name the validation is recorded
   (`git config user.name` as the default; several names allowed; no meaningful name → no
   line).
4. Whether the person considers this capability sufficiently validated. A no leaves the
   candidate where it is: say what remains and that the next run resumes it.

A question states the behaviour in the present tense and asks whether that is how the person
knows it. An `[Uncertain]` statement adds one plain clause on what makes it uncertain — "it
depends on a setting outside the app", never which one. A statement, a conflict, and the
follow-up after a bare "not correct":

```text
Today, a member with a loan 21 days overdue cannot borrow another book. Is that how you
know it?
Today, a reservation lapses after 7 days; a test expects 10. Is 7 days how you know it?
What does the system do instead, as you know it?
```

**Check every question before sending it**, and rewrite it until all three hold:

- It describes what the system does now — nothing about what it should do.
- Someone who knows the system from using, supporting or owning it can answer it without
  reading code: no file path, class, storage, library, setting name, status code or retry.
- It asks for no value, message, limit, time span, rule or decision the system does not
  already have — such a question is a finding, or it belongs to `/sdd-specify`.

**Apply every answer to the candidate at once:**

- *Correct* → `[Confirmed]`.
- *Don't know* → `[Uncertain]`, whatever the marker was, with `reviewed` after it.
- *Not correct* → unless the person already said, ask once what the system does instead, as
  they know it. Treat that as a claim about today and look for it in the code — not only at
  the cited pointer. A line that supports it: you misread — fix the statement,
  `[Confirmed]`. None: remove the statement (its ID is never reused) and record a *Finding*
  with both sides — "the system does X (pointer); <who> knows it as Y, or does not
  recognise it; AC-009 removed". A belief never becomes a rule the code does not have.
- Not business behaviour at all, says the person → remove it (its ID never reused) with one
  *Removed* line naming the ID and why.
- The person says what the system should do → verbatim as a *Wish*, attributed: input for
  `/sdd-specify`, never a rule here, no follow-up. A reason the person volunteers stays with
  its rule, attributed; never ask for reasons.
- A reply to a finding never changes what it is: keep the comment beside it, attributed. If
  the person calls it wrong, check its evidence; drop it only if you misread.

Nothing from this dialogue goes into `AGENTS.md`: a reminder to stay with business behaviour
restates this command.

## Phase D — Promotion to a normal Baseline spec

Precondition: no unreviewed `[Inferred]` or `[Uncertain]` remains — validate it now, or
the user moves it to *Open points* as reviewed and unresolved. Then propose all writes in
**one** confirmation and act only on a yes:

- Write `.specs/done/NNNN-<slug>.md` (next free number across `.specs/`). Header, restating
  `.claude/rules/specs.md`: `**Status:** Baseline` · `**Plan:** _none_` · `**Origin:**
  Reverse — Brownfield` (or `Reverse — Technical Plan`) · `**Validated by:** <names> ·
  <date>`. `Origin` is provenance only — from here on it is a normal spec.
- Markers disappear when everything is confirmed. Reviewed, unresolved uncertainty stays
  visible under *Open points*, each line keeping `[Uncertain]`. A conflict still open becomes
  a *Finding*; findings and wishes stay as written — a Baseline also says what surprised.
- Replace `## Evidence` and `## Conflicts` by `## Technical Reference`, **≤ 8 lines**, paths
  and symbols only: architecture reference (`AGENTS.md` → module) · primary implementation
  (a path pattern) · main contract · deciding tests · in plan mode the historical plan ·
  `analysed at <hash> · <date>`. No directory trees, no code walkthroughs, no invented
  design history. The spec must read complete without the reports.
- **Never write a plan**, never touch `backlog/` or `active/`: a Baseline lives in `done/`
  without one. A later change runs the normal flow — `/sdd-plan` reads the Technical Reference.
- Delete this capability's workflow files by their exact paths —
  `.sdd-reverse/candidates/<slug>.reverse-spec.md` and each
  `.sdd-reverse/reports/<slug>.<lens>.md` — listing the folder rather than searching it:
  editors often leave git-ignored folders out of search. Set the row to `promoted` with the
  spec path, add one line to `.memory-bank/activeContext.md`.

Hand off: name the file and the findings and wishes it carries — changing any of them starts
with `/sdd-specify`, which asks what should be. Recommend `/sdd-clarify` on the Baseline in a
fresh context: a stranger's read finds wording that misleads. Propose the commit (Ask-first,
per `AGENTS.md`).

## Cleanup

When no row is `pending`, `analyzed` or `validating`, ask once, in `DocLanguage`: "Delete
the raw analysis in `.sdd-reverse/reports/`? (recommended)" — and, when no `deferred` row
remains, the whole `.sdd-reverse/` folder. Deferred rows keep `_context.md` and
`_worklist.md` alive: they are the memory of what was seen and not yet reconstructed.
