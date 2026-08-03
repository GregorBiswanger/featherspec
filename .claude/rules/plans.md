---
paths:
  - .specs/**/*.plan.md
---

# Plan file rules

The plan artifact itself — one plan per spec, the `.plan.md` naming, the status vocabulary, and
the duty to keep it current — is defined in `AGENTS.md` (loaded every session). These rules
cover only what is specific to **working inside a plan file**:

- Write in the language specified by `DocLanguage` in `AGENTS.md`.
- Steps stay baby steps: one concern, finishable in one sitting, with a `Verify:` line that can
  actually be run or observed. A step you cannot verify on its own is a step to split.
- **Update the file the moment a step's state changes**, in the same change set as the code:
  tick the checkbox, fill `Notes`, move `Current step`, refresh `Last updated`, and rewrite
  `Session handoff` so a fresh session can continue from the file alone.
- **Fill the traceability table with real paths** (`src/auth/login.ts`, or `file:symbol`) as
  soon as a step lands. Never write a path that does not exist yet — that is what `Do:` is for.
- Record deviations in the step's `Notes:`. A plan that quietly drifts from the code is worse
  than no plan.
- Keep step IDs stable. They are referenced from commits, reports, and the traceability table;
  append new ones instead of renumbering, and strike obsolete steps with a reason.
- Research entries keep their link and retrieval date, so a later session can tell fresh
  findings from stale ones.
- It stays a working document, not a diary: current state and the decisions that still matter.
