---
name: sdd-style-update
description: Capture or reorganize coding style preferences in AGENTS.md.
argument-hint: "[preference] (or list several)"
disable-model-invocation: true
---

<!-- Shared skill for Claude Code and GitHub Copilot. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither, and this one file
     is read by both tools. -->

# /sdd-style-update — Capture Coding Style Preferences

The user may state one or more preferences after the command. If none are given, ask for them.

## Goal

Update the *Style & Output Preferences* section in `AGENTS.md` — the **only** place style
preferences live. No loader file holds a copy.

## Steps

1. If no preference was provided, ask the user for it.
2. Normalize each into a short bullet, grouped by category (Comments, Naming, Formatting,
   Framework Conventions, Testing, etc.).
3. Apply the updates immediately to the *Style & Output Preferences* section in `AGENTS.md`
   (append under the right heading, or create the heading).
4. Confirm by showing the updated bullets. Apply them to all future code generation.
