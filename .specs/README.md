# Specs

Specs live under `.specs/` and are organized by lifecycle stage:

- `.specs/backlog/` — ideas and not-yet-started specs
- `.specs/active/` — specs currently being implemented
- `.specs/done/` — implemented specs with passing acceptance criteria

Each spec declares its status near the top:

`**Status:** Draft | In Progress | Implemented | Deprecated`

A spec exists in **exactly one** lifecycle folder at a time. Use `/sdd-lifecycle` to move
specs and update their status; it deletes the original after moving so no duplicates remain.

The documentation language is controlled by `DocLanguage` in `AGENTS.md`
(default English until `/sdd-setup` sets it).
