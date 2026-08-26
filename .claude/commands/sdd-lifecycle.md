---
description: Manage spec status and move specs between backlog/active/done.
argument-hint: "[spec path] [newStatus — vocabulary in AGENTS.md]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-lifecycle workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-lifecycle.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-lifecycle — Manage Spec Status and Folders

The user may name a spec path and/or a new status after the command. If the spec is not
given, list the contents of `.specs/backlog/`, `.specs/active/`, and `.specs/done/` and ask
which spec to act on.

## Goal

Keep the spec set tidy: update status fields, move specs between `backlog/`, `active/`, and
`done/`, and keep the Memory Bank aligned with important spec changes.

## Rules

> These restate the *Spec & plan lifecycle* policy from `AGENTS.md` on purpose: this command is the one
> that performs the moves and deletions, so the safety rules must be in front of the model at
> the moment it acts. `AGENTS.md` stays authoritative — if the two ever diverge, follow it and
> fix this list.

- Spec files are Markdown, written in the language set by `DocLanguage` in `AGENTS.md`.
- Lifecycle folders: `.specs/backlog/` (ideas), `.specs/active/` (in progress),
  `.specs/done/` (implemented, acceptance criteria satisfied).
- Each spec declares a status near the top:
  `**Status:** Draft | In Progress | Implemented | Deprecated | Baseline`.
- Only move a spec to `done/` when its acceptance criteria are satisfied and tests pass.
- **No duplicates:** a spec exists in exactly one lifecycle folder. When moving, write the
  file to the destination **and delete the original**.
- **Duplicate check:** before moving, scan all three folders for files with the same name.
  If found, keep the copy holding the current working state (normally the one being moved),
  delete the rest — if unclear which is current, ask. Then move.
- **Plans travel with their spec:** if `NNNN-slug.plan.md` exists next to `NNNN-slug.md`, move
  and de-duplicate both together — a plan must never end up in a different folder than its
  spec. Spec and plan have separate status vocabularies; do not overwrite one with the other.
- **Before `done/`:** check the plan too — every step ticked **and its `Verified:` field
  filled**, the traceability table filled with real code paths and a test per criterion. Ask
  for the evidence: a `/sdd-compile` brief whose verdict is `READY`, or the `Verified:` lines
  themselves. Ticked boxes with no recorded run are not evidence — name the criteria that lack
  it and stop. If steps are still open, say so and let the user decide before moving.
  Scan every step's `Notes:` for deviations: one that changed behaviour must be reconciled
  into the spec (updated, or recorded there as accepted) before the move. That reconciliation
  edit is the one sanctioned spec change here — confirm it with the user. A `NOT READY` brief
  blocks the move unless its only blockers are gaps these gates fix in this same run; fix,
  re-verify, then proceed.
- **Deprecated:** the spec stays in `done/` and links its successor spec (or
  `successor: none — behaviour removed`); its plan keeps its status plus an abandonment note.
- **Reactivating an `Implemented` spec** whose behaviour is changing: pair moves back to
  `active/`, spec and plan both `In Progress`; `/sdd-plan` Mode C extends the plan from its
  impact report. A new slice of work gets a successor spec instead — when unsure which case
  it is, ask. `Deprecated` stays reserved for behaviour a successor replaces or removes.
- **Abandoning a spec that was never implemented:** delete it only on the user's instruction
  and note it in `activeContext.md` — no `Deprecated`, no move to `done/`.
- **`Baseline` specs** (existing behaviour, brownfield) live in `done/` without a plan and are
  exempt from the evidence gate — but require the `/sdd-clarify` pass noted in the spec
  (see `/sdd-specify`, Baseline mode).
- **No plan beside the spec?** Ask why and record the answer in the spec. `Baseline` needs no
  plan; for anything else, skipping the plan is the user's recorded decision, a forgotten plan
  is not. (The fast path in `AGENTS.md` means no spec *and* no plan — it does not apply to a
  specced change.)

## Default behavior

1. **Inspect** the referenced spec(s): title, summary, acceptance criteria, current status, and
   the state of the accompanying plan if there is one.
2. **Propose** a lifecycle update (draft → `backlog/`; in progress → `active/`; completed →
   `done/` with status `Implemented`; invalidated → `Deprecated`, stays in `done/` with a
   successor link; changing again → reactivation back to `active/`).
3. **Act**: edit the `**Status:**` line, move the file — together with its `.plan.md` sibling —
   to the target folder, and delete the originals from the source folder.
4. **Sync docs**: update the relevant `.memory-bank/*` files. **Always** update
   `.memory-bank/activeContext.md` when a spec becomes active or is completed — set
   `## Active Spec` (or clear it when moving to `done/`), update `Current phase`, and refresh
   `## Next`. Keep within the size limit from `AGENTS.md`. Close plan handoff lines the move
   made stale. Then propose one commit covering move + sync (Ask-first gate).

## Do / Don't

**Do** — keep changes minimal and focused on lifecycle; preserve spec structure and wording;
mention which specs moved and how their status changed.
**Don't** — change a spec's technical content unless asked; create or remove specs unbidden;
modify code outside spec and Memory Bank files unless asked.
