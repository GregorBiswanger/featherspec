---
description: Capture or reorganize coding style preferences in AGENTS.md.
argument-hint: "[preference] (or list several)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-style-update workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the one-line loader in
     .github/prompts/sdd-style-update.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

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
