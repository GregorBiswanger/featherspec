---
paths:
  - AGENTS.md
---

# Constitution file rules

`AGENTS.md` is the single, tool-neutral source of truth. When editing it:

- Keep it short, structured, and easy to scan (target: under 200 lines). *Style & Output
  Preferences* is the one section meant to grow, so keep room for it. When the file reaches the
  cap, evict in this order: (1) anything a command already carries and only needs at the moment
  that command runs, (2) explanatory prose around a rule — keep the rule, drop the essay,
  (3) examples, once one remains. Never evict a rule to a loader file: that is the drift this
  design exists to prevent. And never evict a sentence that states a gate, an ask-first
  boundary or a non-negotiable — a gate that exists only sometimes is not a gate. If nothing is left to evict, the constitution has outgrown its job
  and the surplus belongs in a path-scoped rule under `.claude/rules/`.
- `DocLanguage`, the `architecture:` snapshot, and *Style & Output Preferences* live **only**
  here. Never copy them into `CLAUDE.md` or any other loader.
- Maintain *Style & Output Preferences* as a living record.
- Keep the `architecture:` snapshot synchronized with the real repo structure.
- `DocLanguage` controls the language of project documentation (Memory Bank + specs + README),
  not this template's own wiring.
