---
paths:
  - .memory-bank/systemPatterns.md
---

# Knowledge records — what the system does, and why it was meant to

No `fs-knowledge:` block and no `provenance:` suffix in the file you are reading? Nothing below
applies. **Evidence establishes what the system does; provenance establishes why it was meant
that way; missing intent stays unknown until knowing it actually matters.**

## The record

Records live under one `## Knowledge records` heading in `.memory-bank/systemPatterns.md`.
The section opens with this legend, once, in `DocLanguage` — a reader must not need a manual:

> *Observation* = what the code does, with a path to check it. *Reason* = why it was built that
> way. `decided` = a trusted source says so, and the source is named. `unknown` = nobody wrote it
> down; a normal, permanent state, not a defect. `candidates` = explanations found somewhere that
> prove nothing. `conflict` = two trusted sources disagree, and neither was chosen.

Each record is a **heading a human can read** plus one small block. The heading *is* the
observation — never repeat it inside the block:

```markdown
### Order events and the outbox row are written in one transaction

```yaml
fs-knowledge:
  id: arch-outbox-001
  evidence: [src/modules/orders/OrderService.ts, src/infrastructure/outbox/OutboxDispatcher.ts]
  rationale:
    state: unknown              # decided | unknown — nothing else
    candidates:                 # optional: found somewhere, proves nothing
      - {statement: "Delivery survives a broker outage.", source: ".specs/done/0001-baseline.md"}
    # conflict: [{statement, source}, …]  · deferred: {at: <date>}
    # a decided reason instead: statement + provenance {type, source|role, origin, confirmedAt}
```
```

Headings and statements are in `DocLanguage`; field names stay English, so the shape survives a
language change. `evidence` needs ≥ 1 path (optionally `path:symbol`), or
`"absent: <path> (<document naming it>)"` where the observation *is* that documented material has
no counterpart in code. ≤ 10 block lines. Other artifacts carry the **id**, never a copy, and no
record ever goes into a file that loads every session.

**One shape, one place: visible in the code → record · not yet in the code → bullet.** Everything
you can *see* becomes a record, whatever its reason state. An intent with no code counterpart yet
stays a dated *Key decisions* bullet with a `provenance: human (<workflow>)` suffix. Never both,
never a third form.

## What may become `decided`

Only three sources, and each names itself in `provenance`:

1. **`human`** — the user confirmed this exact statement in a workflow. Also record
   `origin: stated` (they formulated it) or `origin: suggested` (they picked an option you wrote),
   and `role` where they name one. A role is metadata, never permission.
2. **`adr`** · 3. **`spec`** — a decision document, with its path, that passes the trust test.

**Trust test.** A document *you did not write* (ADR, requirement doc — outside `.specs/`,
`.memory-bank/`, `.architecture/`, `.sdd-*/`) is trusted when it states the rationale in its own
text, its status is live (`Superseded`, `Rejected`, `Deprecated` are not) and it does not declare
itself generated. In a document *FeatherSpec wrote*, the **statement** is the unit, not the file:
only one carrying an explicit human-provenance marker is trusted, whatever the file's status — an
`Implemented` spec was accepted as a description of what to build, not sentence by sentence as a
record of why.

**Never `decided`:** inference, pattern recognition, naming, "this pattern is usually for…",
best practice, a code comment, a README, your own earlier output. A source-backed explanation
becomes a `candidate` with its source. An explanation with no source is not written down at all.

## When to persist, and when to ask

Persist a record only when a change to its subject would be decided differently depending on the
answer — **and** the rationale is `decided` but not recoverable from the evidence files, or
`unknown` in a way that could block a future decision, or contested by two trusted sources.
Wiring, validation and logging never qualify.

**Ask only** when the change in front of you touches the record's subject **and** the repository
does not settle the constraint **and** two options differ in an observable property — behaviour,
a guarantee, a contract, what data survives — **and** the unknown reason could decide between
them. Plus one override: a change that may alter a **security, compliance or data-integrity**
guarantee while the repository does not say which guarantee must hold. Category alone never asks.

A question states: what you verified · what you could not · why it matters for *this* decision ·
which decision or guarantee it affects — then offers options, always one of which answers nothing
and defers. Never ask "why does X exist?". Blocking blocks the one decision, never the run: write
the open decision down and carry on. Deferred once = not asked again this run; a later ask names
the earlier deferral. Only the user retracts a `decided`. Never repair or delete a record you
cannot parse — report it.
