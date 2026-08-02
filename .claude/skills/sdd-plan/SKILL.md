---
name: sdd-plan
description: Create a step-by-step implementation plan for a spec.
argument-hint: "[path-to-spec.md]"
disable-model-invocation: true
---

<!-- Shared skill for Claude Code and GitHub Copilot. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither, and this one file
     is read by both tools. -->

# /sdd-plan — Implementation Plan

The user may name a spec path after the command. If none is given, list the specs under
`.specs/` and ask which one to plan.

- Read the relevant spec under `.specs/`.
- Propose a step-by-step plan with checkpoints, each small and verifiable.
- Identify which Memory Bank docs (`.memory-bank/*`) will change.
- Note the acceptance criteria each step satisfies.
- Write the reply in `DocLanguage`. Do not change code yet — this skill only plans.
