---
name: sdd-lifecycle
description: Manage spec status and move specs between backlog/active/done.
argument-hint: "[spec path] [newStatus: Draft|In Progress|Implemented|Deprecated]"
disable-model-invocation: true
---

<!-- Shared skill for Claude Code and GitHub Copilot. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither, and this one file
     is read by both tools. -->

# /sdd-lifecycle — Manage Spec Status and Folders

The user may name a spec path and/or a new status after the command. If the spec is not
given, list the contents of `.specs/backlog/`, `.specs/active/`, and `.specs/done/` and ask
which spec to act on.

## Goal

Keep the spec set tidy: update status fields, move specs between `backlog/`, `active/`, and
`done/`, and keep the Memory Bank aligned with important spec changes.

## Rules

> These restate the *Spec lifecycle* policy from `AGENTS.md` on purpose: this skill is the one
> that performs the moves and deletions, so the safety rules must be in front of the model at
> the moment it acts. `AGENTS.md` stays authoritative — if the two ever diverge, follow it and
> fix this list.

- Spec files are Markdown, written in the language set by `DocLanguage` in `AGENTS.md`.
- Lifecycle folders: `.specs/backlog/` (ideas), `.specs/active/` (in progress),
  `.specs/done/` (implemented, acceptance criteria satisfied).
- Each spec declares a status near the top: `**Status:** Draft | In Progress | Implemented | Deprecated`.
- Only move a spec to `done/` when its acceptance criteria are satisfied and tests pass.
- **No duplicates:** a spec exists in exactly one lifecycle folder. When moving, write the
  file to the destination **and delete the original**.
- **Duplicate check:** before moving, scan all three folders for files with the same name.
  If found, delete copies in less-advanced folders (done > active > backlog), then move.

## Default behavior

1. **Inspect** the referenced spec(s): title, summary, acceptance criteria, current status.
2. **Propose** a lifecycle update (draft → `backlog/`; in progress → `active/`; completed →
   `done/` with status `Implemented`).
3. **Act**: edit the `**Status:**` line, move the file to the target folder, and delete the
   original from the source folder.
4. **Sync docs**: update the relevant `.memory-bank/*` files. **Always** update
   `.memory-bank/activeContext.md` when a spec becomes active or is completed — set
   `## Active Spec` (or clear it when moving to `done/`), update `Current phase`, and refresh
   `## Next`. Keep it to 1–2 screen pages.

## Do / Don't

**Do** — keep changes minimal and focused on lifecycle; preserve spec structure and wording;
mention which specs moved and how their status changed.
**Don't** — change a spec's technical content unless asked; create or remove specs unbidden;
modify code outside spec and Memory Bank files unless asked.
