---
name: sdd-architecture-update
description: Detect architecture drift and sync the snapshot + Memory Bank (with confirmation).
argument-hint: "[focus: module|folder|area] (optional)"
disable-model-invocation: true
---

<!-- Shared skill for Claude Code and GitHub Copilot. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither, and this one file
     is read by both tools. -->

# /sdd-architecture-update — Architecture Reconciliation

The user may name a focus area (module, folder, or concern) after the command; it is optional.

## Gather context first

Inspect the actual repository directory tree (top two levels are enough) and read the current
`architecture:` snapshot from `AGENTS.md`.

## Goal

Compare the actual repository structure with the `architecture:` snapshot in `AGENTS.md` and
reconcile them.

## Steps

1. Read the tree and the current snapshot.
2. Identify changes that matter architecturally: new/moved/removed top-level folders; new
   entrypoints, apps, services, packages, modules; changed boundaries or shared components.
3. Present a **delta report**: what changed (observed), why it matters (brief), and the
   proposed updates to the snapshot and Memory Bank docs.

## Confirmation gate

Before writing anything, ask:

- "Is this the change you expected?"
- "Anything else I should include (e.g., context not visible in the code)?"

Only after confirmation:

- Update the `architecture:` YAML in `AGENTS.md` (the only place it lives).
- Update `.memory-bank/systemPatterns.md` (patterns/decisions).
- Update `.memory-bank/activeContext.md` (recent changes + next steps; keep to 1–2 pages).

Everything must remain **DocLanguage-aware**.
