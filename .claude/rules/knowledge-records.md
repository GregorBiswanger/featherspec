---
paths:
  - .memory-bank/systemPatterns.md
---

# Knowledge records — what the system does, and why it was meant to

No `fs-knowledge:` block and no `provenance:` suffix in the file you are reading? Nothing below
applies. **Evidence establishes what the system does; provenance establishes why it was meant
that way; missing intent stays unknown until knowing it actually matters.**

## The record

```yaml
fs-knowledge:
  id: arch-outbox-001                       # stable, never reused, unique in the project
  observation:
    statement: "Order events are written to an outbox table inside the business transaction."
    evidence:                               # ≥ 1 path (optionally :symbol), or "absent: <path> (<doc>)"
      - src/infrastructure/outbox/OutboxDispatcher.ts
  rationale:
    state: unknown                          # decided | unknown — nothing else
    candidates:                             # optional, while unknown: source-backed but untrusted
      - {statement: "The feed refreshes once a minute.", source: "src/shared/cache/TtlCache.ts:7"}
    # conflict:  [{statement, source}, …]   two trusted sources disagreeing — keep both, pick neither
    # deferred:  {at: 2026-08-29}           the user postponed the question
    # retracted: {statement, provenance, at}  a confirmation the user withdrew
```

A `decided` rationale replaces `state: unknown` with three lines:

```yaml
    state: decided
    statement: "Finance must reconstruct an order's exact state at the time it was paid."
    provenance: {type: adr, source: docs/adr/0001-event-sourcing.md, confirmedAt: 2026-03-14}
```

≤ 12 lines per record, one fenced block each, in `.memory-bank/systemPatterns.md` only. Other
artifacts carry the **id** and never a copy. Never in a file that loads every session.

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
which decision or guarantee it affects — then offers options, always including one that answers
nothing and defers. Never ask "why does X exist?". Blocking blocks the one decision, never the
run: state the open decision in what you write and carry on with everything else.

Deferred once = not asked again this run; a later ask names the earlier deferral. Only the user
retracts a `decided`. Never repair or delete a record you cannot parse — report it.
