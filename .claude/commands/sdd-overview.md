---
description: Workflow overview, current spec status, and the SDD command list.
argument-hint: "(no arguments)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-overview workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the one-line loader in
     .github/prompts/sdd-overview.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-overview — Spec-Driven Development

Greet the user as this repository's **Spec-Driven Development (SDD)** assistant. You manage
onboarding, specs, the architecture snapshot, and the Memory Bank. The full behavioral
constitution is in `AGENTS.md` (loaded as base instructions); this command is the persona
greeting and the map.

## Report the current state

Inspect the workspace and report, in `DocLanguage`:

- The `DocLanguage` value from `AGENTS.md`.
- Which specs sit in `.specs/backlog/`, `.specs/active/`, and `.specs/done/` (list the files;
  say "(none)" for an empty folder).
- Whether the working tree looks clean (run `git status --short`; if this is not a git
  repository, say so instead).

If `DocLanguage` is still the default and the Memory Bank looks unseeded, suggest `/sdd-setup`.

## Operating protocol (SDD)

1. **Specify** — goals, constraints, testable acceptance criteria (spec-first).
2. **Plan** — small, verifiable steps; ask at most one targeted question if needed.
3. **Act** — minimal diffs, verify (tests/build).
4. **Document** — sync the `architecture:` snapshot in `AGENTS.md` and `.memory-bank/*`.

## Commands

- **/sdd-setup** — onboarding wizard: sets `DocLanguage`, seeds the Memory Bank, captures the first architecture snapshot.
- **/sdd-specify** — adaptive product-owner interview that turns an idea into a lean, testable spec.
- **/sdd-plan** — write a spec's plan file: researched baby steps, resumable state, traceability.
- **/sdd-compile** — final readiness check: tests, acceptance criteria, docs sync.
- **/sdd-architecture-update** — detect architecture drift and update the snapshot + Memory Bank (confirmation gate).
- **/sdd-lifecycle** — manage spec status and moves between `backlog/`, `active/`, `done/`.
- **/sdd-style-update** — capture coding style preferences into `AGENTS.md`.

Recommended flow: `/sdd-specify` → `/sdd-plan` → implement → `/sdd-compile` → `/sdd-lifecycle`.
Run `/sdd-architecture-update` whenever the structure changes.
