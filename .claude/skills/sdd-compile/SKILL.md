---
name: sdd-compile
description: Final readiness check — tests, acceptance criteria, docs sync.
argument-hint: "[path-to-spec.md] [runTests:true|false]"
disable-model-invocation: true
---

<!-- Shared skill for Claude Code and GitHub Copilot. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither, and this one file
     is read by both tools. -->

# /sdd-compile — Execution Brief

The user may name a spec path and whether to run tests after the command. If the spec is not
given, ask which one; default to running tests when the project clearly has a test command.

## Gather context first

Run `git status --short` and `git log --oneline -10` (if this is a git repository) and use
the results. Read the referenced spec and `AGENTS.md`.

## Produce a concise readiness brief

- **Current goal** — from the referenced spec.
- **Constraints** — from `AGENTS.md` and the spec.
- **Architecture snapshot highlights** — relevant parts of the `architecture:` block.
- **Acceptance criteria** — each one, marked satisfied / pending, with evidence.
- **Do / Don't** — including the *Style & Output Preferences* from `AGENTS.md`.
- **Docs sync** — is the Memory Bank current? Does the spec status match reality?
- **Next 3 steps** — concrete and actionable.

If tests should run, run them and report results. Write the brief in `DocLanguage`.
