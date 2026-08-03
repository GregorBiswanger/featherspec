# Specs

Specs live here, one folder per lifecycle stage:

- `.specs/backlog/` — ideas and not-yet-started specs
- `.specs/active/` — specs currently being implemented
- `.specs/done/` — implemented specs with passing acceptance criteria

A planned spec keeps its plan beside it as `<same-name>.plan.md` — the persisted step list and
the traceability from acceptance criteria to code. The two files move together.

The normative rules — the status vocabularies, the one-folder-at-a-time rule, and the move
procedure — are defined in [`AGENTS.md`](../AGENTS.md) under *Spec & plan lifecycle*. Do not
restate them here; that file is the single source of truth and is loaded in every session.

Use `/sdd-specify` to create a spec, `/sdd-plan` to plan it, and `/sdd-lifecycle` to change its
status or move it.
