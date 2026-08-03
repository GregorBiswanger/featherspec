---
paths:
  - .specs/**/*.md
---

# Specs writing rules

The spec lifecycle itself — folder meanings, the status vocabulary, and the move procedure —
is defined in `AGENTS.md` (loaded every session). These rules cover only what is specific to
**writing a spec file**:

- Write specs in the language specified by `DocLanguage` in `AGENTS.md`.
- Keep acceptance criteria explicit and testable.
- Acceptance criteria carry stable IDs (`AC-NNN`); never renumber them — steps and tests
  reference them.
- Binding criteria use **shall**. Reject any criterion containing: typically · usually ·
  appropriate · sufficient · performant · user-friendly · fast · robust · as needed · etc. ·
  the literal token "and/or" (plain "or" enumerations are fine) — or a passive verb with no
  actor. A criterion nothing can prove false is not a criterion.
- State the spec's status on a `**Status:**` line near the top, using the vocabulary from
  `AGENTS.md`.

Plan files (`.specs/**/*.plan.md`) have their own craft rules in `plans.md` — status
vocabulary, step upkeep, and traceability live there, not here.

> Note: path-scoped rules load when a matching file is **read**. When `/sdd-specify` creates
> a brand-new spec, this rule may not be loaded yet — which is why `/sdd-specify` restates
> the essentials (DocLanguage, the `**Status:**` line, the document structure) inline.
