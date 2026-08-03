---
description: Adversarial pass over a finished spec — contradictions, ambiguity, untestable criteria, missing failure modes.
argument-hint: "[path to spec]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-clarify workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the one-line loader in
     .github/prompts/sdd-clarify.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-clarify — Adversarial Spec Review

The user may name a spec path after the command. If none is given, list what sits under
`.specs/` and ask which one to review.

## Read it as a stranger

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

Produce exactly four lists, in `DocLanguage`. Quote the spec. An empty list is a real and
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
criterion carrying a rejected word (typically · usually · appropriate · sufficient · performant
· user-friendly · fast · robust · as needed · etc. · and/or) or a passive verb with no actor.

### 4. Failure modes the spec never names

What happens when input is missing, malformed, duplicated, or arrives twice? When a dependency
is unreachable, slow, or returns something unexpected? When two users act at once? When the
operation half-succeeds? Report only the ones that matter for *this* spec — an invented edge
case costs more than a missed one.

## Close with one question

End with a single question: the item whose being wrong would cost the most. One question, not a
catalogue — a catalogue gets skimmed and answered in bulk, which is the same as not asking.

Then say plainly whether the spec is safe to plan from as it stands, or name what must be
settled first. Do not edit the spec: unresolved items belong in its *Open points* section, and
that is the user's call to make with `/sdd-specify` or by hand.
