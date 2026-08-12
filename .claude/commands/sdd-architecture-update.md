---
description: Detect architecture drift and sync the snapshot + Memory Bank (with confirmation).
argument-hint: "[focus: module|folder|area] (optional)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-architecture-update workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-architecture-update.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-architecture-update — Architecture Reconciliation

The user may name a focus area (module, folder, or concern) after the command; it is optional.

## Gather context first

Read the current `architecture:` snapshot from `AGENTS.md`. Inspect the tree's top two levels
for new structures, and verify every path the snapshot names (entrypoints, modules, shared) at
whatever depth it sits — boundary changes hide below level two.

## Goal

Compare the actual repository structure with the `architecture:` snapshot in `AGENTS.md` and
reconcile them.

## Steps

1. Read the tree and the current snapshot.
2. Identify changes that matter architecturally: new/moved/removed top-level folders; new
   entrypoints, apps, services, packages, modules; changed boundaries or shared components.
3. Present a **delta report**: what changed (observed), why it matters (brief), and the
   proposed updates to the snapshot and Memory Bank docs.

## When reconciliation is not enough

If the tree shows several structures the snapshot cannot place, the snapshot names paths that
no longer exist, or the focus area sits under `unmapped:` — do not guess. Recommend
`/sdd-architecture-scan` (with the focus path) in the delta report instead. (This spells out
the escalation rule from `AGENTS.md` for the moment this command acts; `AGENTS.md` stays
authoritative.)

## Confirmation gate

Before writing anything, ask:

- "Is this the change you expected?"
- "Anything else I should include (e.g., context not visible in the code)?"

Only after confirmation:

- Update the `architecture:` YAML in `AGENTS.md` (the only place it lives), and set the
  `# last reconciled:` comment inside that block to today's date — a snapshot with no date
  cannot be told apart from one that was never checked.
- When `.architecture/` exists, update the affected module map(s) and their
  `# last reconciled:` lines in the same change set.
- Update `.memory-bank/systemPatterns.md` (patterns/decisions).
- Update `.memory-bank/techContext.md` when stack, build or test facts changed.
- Update `.memory-bank/activeContext.md` (recent changes + next steps; size limit from
  `AGENTS.md`).
- Measure `AGENTS.md` against the cap in `.claude/rules/constitution.md`; if over, propose an
  eviction per its order.

## When invoked from /sdd-architecture-scan

Treat the scan's distilled findings as the observed state; include its coverage figures and
navigation self-test score in the delta report; on confirmation also set `last deep scan` in
the snapshot comment. The gate above stays the snapshot's only write gate. If the findings
arrive without a recorded self-test score, do not open the gate — send the scan back to run
its self-test first: an unverified fingerprint is not observed state.

Everything must remain **DocLanguage-aware**.
