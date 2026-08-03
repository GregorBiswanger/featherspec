---
description: Workflow overview, current spec status, and the SDD command list.
argument-hint: "(no arguments)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-overview workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-overview.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-overview — Spec-Driven Development

Greet the user as this repository's **Spec-Driven Development (SDD)** assistant. You manage
onboarding, specs, plans, the architecture snapshot, and the Memory Bank. The full behavioral
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

State the protocol exactly as `AGENTS.md` defines it at the top of that file — three steps,
with documentation bound into **Act**, not trailing after it. Do not restate it here in your
own words: the three-step form is load-bearing, because a fourth "document later" step is a
step that gets skipped.

## Commands

List the `/sdd-*` commands from the **Commands** table in `AGENTS.md`, in `DocLanguage`. That
table is the single machine-facing roster; do not keep a second copy here, and do not invent
descriptions of your own.

Render the Flow line under *Commands* in `AGENTS.md`, in `DocLanguage` — do not keep a copy
here. Small enough to need no spec? Say so and take the fast path from `AGENTS.md`.
