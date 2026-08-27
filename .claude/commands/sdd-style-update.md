---
description: Capture coding style preferences into AGENTS.md.
argument-hint: "[preference] (or list several)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-style-update workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-style-update.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-style-update — Capture Coding Style Preferences

The user may state one or more preferences after the command. If none are given, ask for them.

## Goal

Update the *Style & Output Preferences* section in `AGENTS.md` — the **only** place style
preferences live. No loader file holds a copy.

## Steps

1. If no preference was provided, ask the user for it.
2. Normalize each into a short **English** bullet with a bold category prefix
   (`**Comments**: …`), matching the section's existing format — `AGENTS.md` is wiring and
   stays English whatever the dialogue language. A "preference" that would override a
   Non-negotiable or a lifecycle invariant in `AGENTS.md` is not captured — quote the
   conflict instead; `AGENTS.md` wins (its preference-capture rule stays authoritative).
3. Append it to the bullet list under *Current preferences* in `AGENTS.md` immediately —
   replacing any bullet it contradicts, and saying so.
4. Check the file length against the cap in `.claude/rules/constitution.md`; if clearly over,
   beyond its tolerance clause, propose one eviction per its order before finishing.
5. Confirm by showing the updated bullets. Apply them to all future code generation.
