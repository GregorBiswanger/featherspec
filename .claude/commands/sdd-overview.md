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
  say "(none)" for an empty folder). While listing, flag as a warning: a file name that
  appears in more than one lifecycle folder; a `.plan.md` in `backlog/` or `active/` with no
  same-name spec beside it, or whose stem already has a dated twin in `.specs/plan-archive/`
  (a moved file resurrected by a later editor save); a `done/` spec with a same-name
  `.plan.md` still beside it (pre-1.5 layout — `/sdd-lifecycle` archives it in passing);
  and a `done/` spec that is not `Baseline` whose `**Plan:**` line names no plan (beside it
  or in `.specs/plan-archive/`). Detection only — the fix belongs to `/sdd-lifecycle`.
- If the snapshot comment names a `last deep scan` date: report it plus the count of `unmapped:` entries.
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
