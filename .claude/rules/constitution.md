---
paths:
  - AGENTS.md
---

# Constitution file rules

`AGENTS.md` is the single, tool-neutral source of truth. When editing it:

- Keep it short, structured, and easy to scan (target: under 200 lines).
- `DocLanguage`, the `architecture:` snapshot, and *Style & Output Preferences* live **only**
  here. Never copy them into `CLAUDE.md` or any other loader.
- Maintain *Style & Output Preferences* as a living record.
- Keep the `architecture:` snapshot synchronized with the real repo structure.
- `DocLanguage` controls the language of project documentation (Memory Bank + specs + README),
  not this template's own wiring.
