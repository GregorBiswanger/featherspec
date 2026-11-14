---
description: Adversarial pass over a finished spec — contradictions, ambiguity, untestable criteria, implementation posing as intent, missing failure modes; then an optional guided walkthrough of the findings.
argument-hint: "[path to spec]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-clarify workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-clarify.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-clarify — Adversarial Spec Review

The user may name a spec path after the command. If none is given — or the named path does
not exist — list what sits under `.specs/` and ask which one to review.

## Read it as a stranger

**If this session wrote or discussed the spec, do not review it here.** Delegate to a
subagent that receives only the spec path and `AGENTS.md`, or tell the user to run this in a
fresh session — in-context knowledge cannot be unread. Only if neither is possible, fall back
to role-play:

**Assume no conversation history.** Whatever was discussed while the spec was written is not
available to you and must not be inferred — if you find yourself filling a gap from context,
that gap is exactly what you are here to report. The context that produced an ambiguity is the
worst placed to find it; your only advantage is that you were not there.

Read the spec, and read `AGENTS.md` for the constraints it must respect. Read the code the spec
touches only when you need it to judge whether a claim is decidable. Do not read the plan — a
plan that already resolved an ambiguity would hide it from you. A finding that rests on
`AGENTS.md` — a setting, a working agreement — is checked against the file on disk before
it is listed: the copy a session started with can predate `/sdd-setup`.

## List first, resolve only with the user

Your job is to find, not to fix. While you list: do not rewrite the spec, do not propose
wording, do not answer your own findings. A model asked to resolve ambiguity resolves it
silently with whatever is statistically plausible; that is the failure this command exists
to catch. Decisions come afterwards, one at a time, and they are the user's (*Walkthrough*).

Produce exactly six lists, in `DocLanguage`. Quote the spec. One or two lines per item — the
quote and what is wrong with it; the reasoning waits for the walkthrough or a request for
details. Short items, complete lists: walk every lens to its end, and never drop a finding
to keep the report short — a small one costs a line here and a wrong guess later. An empty
list is a real and useful answer — say "none found" rather than manufacturing an item.

### 1. Contradictions

Two statements that cannot both hold. Look hardest between sections that are rarely read
together: a business rule against an acceptance criterion, an out-of-scope entry against a
functional requirement, a non-functional limit against the main flow.

### 2. Terms with more than one meaning

Any word the spec uses in two senses, or a domain term it never defines. Name each occurrence
and the readings it allows. "User", "order", "active", "valid" and "sync" are the usual
offenders.

### 3. Criteria nothing can decide

Walk every `AC-`. For each, name the observation that would prove it **false**. If you cannot,
list it — and say whether it is unfixable as written or merely missing a threshold. Flag any
criterion violating the vocabulary rule in `.claude/rules/specs.md` (rejected words, passive
verb with no actor) — that rule is the authoritative list.

### 4. Implementation posing as intent

Requirements or criteria that name a technology, storage or framework where behaviour was
meant — anything that stops being true if the stack is swapped. Quote each; do not rewrite it.

### 5. Failure modes the spec never names

What happens when input is missing, malformed, duplicated, or arrives twice? When a dependency
is unreachable, slow, or returns something unexpected? When two users act at once? When the
operation half-succeeds? Report only the ones that matter for *this* spec — an invented edge
case costs more than a missed one.

### 6. Guesses dressed as decisions

Assumptions whose "why we believe it" column is empty or circular, assumptions that are really
undecided requirements, and *Out of scope* entries missing their "— because" reason or
contradicting a functional requirement.

## Close: count, verdict, one offer

Below the lists give the count — "7 findings, 2 block planning": a finding blocks when a
plan written from the spec as it stands would have to guess. Then say plainly whether the
spec is safe to plan from, or name what must be settled first.

No findings → stop there; there is nothing to walk through. Otherwise end with one offer,
not a catalogue of questions — a catalogue gets skimmed and answered in bulk, which is the
same as not asking: go through the findings one at a time; you bring options and, where
something supports one, a recommendation; the user decides. Name the two other ways out in
the same breath: the details of any finding on request, or the findings appended verbatim,
as open questions, to the spec's *Open points* so they survive this session (append only
after a yes). Where the tool has a question picker, put the offer there: walk through
together, details on a finding, park them in *Open points*, nothing for now. Resolving
stays the user's call — in the walkthrough, with `/sdd-specify` (revise mode) or by hand.

## Walkthrough (only after a yes)

One finding per message, costliest first — every blocking finding before the rest; once
the blocking ones are settled, ask once whether to go on with the others or park them in
*Open points*. Each message: `Question 2 of 7`, the quote, one sentence on what goes wrong
if it stays open, then two to four lettered options that lead to different outcomes, each
with its consequence in a clause. Mark one **recommended** only when the spec, `AGENTS.md`,
the Memory Bank or the code supports it, and name that support; with no basis, say so
instead of recommending what is merely plausible. The user may pick a letter, answer in
their own words, skip the finding, or stop the walkthrough. An answer that is not plainly
one of the options: read it back in one line and wait for the yes. Never decide for the
user, never ask two findings at once. Where the tool has a question picker, use it.

A delegated review hands back its lists, count and verdict; the offer and the walkthrough
then run in the main session — finding needed the stranger, deciding does not. Walk every
finding as it was listed: only the user skips one.

After the last answer, or when the user stops: write the confirmed decisions into the spec
in one pass, by the conventions of `/sdd-specify`'s revise mode (not its interview) — same
file, affected sections only, number and `AC-` IDs stable, each resolved finding recorded in
*Open points* — and append every skipped or unreached finding there as an open question.
Write each decision at the precision it was confirmed: a number, a message or a new
criterion the chosen option did not state is not yours to add — it goes to *Open points*.
Nothing the user did not confirm enters the spec. Close in a few lines: what was decided, what
stays open, whether the spec
is now safe to plan from, and the next command (`/sdd-plan` — it realigns a plan that
already exists).
