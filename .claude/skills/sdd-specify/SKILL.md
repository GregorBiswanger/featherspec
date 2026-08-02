---
name: sdd-specify
description: Turn an idea into a spec skeleton with testable acceptance criteria.
argument-hint: "[title] [area/module]"
disable-model-invocation: true
---

<!-- Shared skill for Claude Code and GitHub Copilot. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither, and this one file
     is read by both tools. -->

# /sdd-specify — Create a Spec

The user may name a title and/or area after the command. If they are missing, gather them in
the questions below.

## Instructions

- Scan `.specs/` for existing spec filenames so you do not collide with or duplicate one.
- Ask for: goal, scope, non-goals, constraints, acceptance criteria, risks — unless the
  provided title/area already make them clear.
- Propose a spec filename under `.specs/backlog/` (a new spec starts in `backlog/`). Use a
  zero-padded numeric prefix and a kebab-case slug, e.g. `0001-user-login.md`, and write it.
- Tell the user to run `/sdd-plan` next.

## Spec essentials (stated here because the `.specs/**` rule may not be loaded yet)

Path-scoped rules load when a matching file is **read**. A brand-new spec has not been read
yet, so restate the essentials here:

- Write the spec in the language set by `DocLanguage` in `AGENTS.md` (default English until `/sdd-setup` sets it).
- Put a status line near the top: `**Status:** Draft`
- Keep acceptance criteria explicit and **testable**.

## Section skeleton

```markdown
# <Title>

**Status:** Draft

## Summary
## Problem / Motivation
## Goals / Non-goals
## Requirements
## Acceptance Criteria (testable)
## Open Questions
## Out of Scope
```
