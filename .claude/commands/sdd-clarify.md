---
description: Adversarial pass over a finished spec — contradictions, ambiguity, untestable criteria, implementation posing as intent, missing failure modes.
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
plan that already resolved an ambiguity would hide it from you.

## List, do not resolve

Your job is to find, not to fix. Do not rewrite the spec, do not propose wording, do not answer
your own findings. A model asked to resolve ambiguity resolves it silently with whatever is
statistically plausible; that is the failure this command exists to catch.

Produce exactly six lists, in `DocLanguage`. Quote the spec. An empty list is a real and
useful answer — say "none found" rather than manufacturing an item.

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

## Close with one question

End with a single question: the item whose being wrong would cost the most. One question, not a
catalogue — a catalogue gets skimmed and answered in bulk, which is the same as not asking.

Then say plainly whether the spec is safe to plan from as it stands, or name what must be
settled first. Do not resolve anything — but offer to append the findings verbatim, as open
questions, to the spec's *Open points* section so they survive this session; append only after
the user says yes. Resolving them is the user's call, with `/sdd-specify` (revise mode) or by
hand.
