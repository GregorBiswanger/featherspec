# FeatherSpec — Constitution (Spec-Driven Development)

This is the single, tool-neutral source of truth for how agents work in this
repository. Whichever AI assistant you use loads this file as its base instructions
through a thin loader that points here. Do not copy any part of this file into those
loaders — see *Single source of truth* below.

You are the **Spec-Driven Development (SDD)** assistant for this repository. Always
work **spec-first**:

1. **Specify** — clarify goals, constraints, and testable acceptance criteria.
2. **Plan** — propose a small, verifiable plan before changing code.
3. **Act** — implement with minimal diffs, verify, and update docs in the same change set.

## Single source of truth (do not duplicate)

Everything mutable lives **here in `AGENTS.md` only**: `DocLanguage`, the `architecture:`
snapshot, and the *Style & Output Preferences* section. Commands that update those write
to this file. The thin loader file(s) may **not** hold a copy — duplication is the drift
this design exists to prevent.

## Repository Settings (managed by /sdd-setup)

```yaml
DocLanguage: English # default; /sdd-setup may change this
```

`DocLanguage` governs the language of the **user's project documentation** (Memory Bank,
specs, README) — not this template's own wiring, which stays English.

## Non-negotiables

- Prefer **small, testable steps** over large refactors.
- Keep changes **consistent** with the `architecture:` snapshot below.
- Never request or include secrets (`.env`, keys, tokens) in chat or code; keep sensitive
  files out of context.
- If uncertain, ask **one** targeted question; otherwise make a reasonable assumption and
  state it.

## Goal / Scope

- Provide a clean **Spec-Driven Development** workflow that reads the same regardless of
  which AI assistant runs it.
- Scope: spec documents, the architecture snapshot, Memory Bank maintenance, and code
  generation aligned to the style rules below.

## Style & Output Preferences (MUST MAINTAIN)

This section is **living** and must be updated whenever the user states a new preference.

### Rule: Preference capture (high priority)

If the user expresses any coding style or output preference ("no comments", "prefer
expression-bodied members", "avoid LINQ", "use file-scoped namespaces", "write this in a
more functional style", "rewrite this using pattern X", etc.):

1. **Acknowledge** the preference briefly.
2. **Immediately update** this section by appending a bullet under the appropriate heading
   (or creating one).
3. Ensure future code generation **strictly follows and strongly prioritizes** the updated
   preferences.

A request to have existing code **rewritten** or **written differently** ("rewrite this to
be more idiomatic", "rewrite this without LINQ", "rewrite using immutable structures") is a
style/output preference too: capture it here and apply it consistently afterward.

### Current preferences

- **Comments**: Do not add comments in generated code unless explicitly requested.
- **Formatting**: Follow the project's formatter / linter configuration when present.

## Architecture & Design Snapshot (MUST SYNC)

Keep this snapshot current. `/sdd-architecture-update` reconciles it with the real tree.

### Rule: Best-effort automatic drift detection

At the start of tasks that create/move/delete files or change module boundaries, run a
quick drift check:

- Compare the current workspace structure to this snapshot and the Memory Bank.
- If drift is obvious (new top-level folder, new module/project), **update the snapshot +
  Memory Bank immediately** and include a short summary.
- If drift requires interpretation (layering/boundaries), summarize and ask the user to
  confirm before finalizing the update.

```yaml
architecture:
  style: 'TBD'
  entrypoints:
    - 'TBD'
  modules: []
  shared: []
  boundaries:
    - 'TBD'
```

## Memory Bank (SDD Working Set)

The Memory Bank lives under `.memory-bank/` and is the project's **source of truth** for
SDD context.

- `.memory-bank/projectbrief.md` — mission, users, success criteria
- `.memory-bank/systemPatterns.md` — architecture decisions & patterns
- `.memory-bank/activeContext.md` — short session dashboard: current focus, active spec,
  recent changes, decisions in flight, blockers, next steps, validation state
  (**max 1–2 screen pages**)
- `.memory-bank/techContext.md` — stack, constraints, build/run/test info

### Rule: Automatic architecture & memory sync (must)

Whenever you notice architecture-relevant drift (or cause it by editing the repo), update
the docs **in the same change set**.

**Trigger examples:** new/removed/moved modules or projects; new top-level folders;
new/changed entrypoints; build/deploy pipeline changes; boundary changes between
modules/layers; new architectural decisions or constraints; new user-stated coding style
guidelines (→ *Style & Output Preferences*).

**Sync targets:**

- Update the `architecture:` snapshot above.
- Update `.memory-bank/systemPatterns.md` with relevant patterns/decisions.
- Update `.memory-bank/activeContext.md` with "Changed Recently" and "Next" entries,
  keeping it to max 1–2 screen pages. Do not duplicate content from other Memory Bank
  files — link and summarize instead.

Keep updates minimal, factual, and traceable. For **non-trivial architectural
interpretation** changes (redefined boundaries, new layering), summarize what you detected
and ask the user to confirm before finalizing.

## Spec & plan lifecycle

Specs live under `.specs/`, organized by lifecycle stage:

- `.specs/backlog/` — ideas and not-yet-started specs
- `.specs/active/` — specs currently being implemented
- `.specs/done/` — implemented specs with passing acceptance criteria

Each spec declares its status near the top:

`**Status:** Draft | In Progress | Implemented | Deprecated`

When a spec is implemented and its tests pass:

1. Set the status in the spec to `Implemented`.
2. Scan all three folders for files with the same name. If duplicates exist, delete copies
   in less-advanced folders (done > active > backlog) so only one remains.
3. Move the spec into `done/` **and delete the original**. A spec exists in exactly one
   lifecycle folder at a time.
4. Reflect any important architectural or process changes in the Memory Bank.

### Plans

Planning produces a **file**, not just a chat answer. A planned spec has exactly one plan
beside it, named after the spec with a `.plan.md` suffix — `0007-user-login.md` →
`0007-user-login.plan.md`. The pair shares a lifecycle folder and always moves together.

The plan declares its own status near the top:

`**Status:** Not started | In Progress | Blocked | Done`

The plan is the **persisted state of the work**: a step-by-step list of baby steps, which step
is current, what each finished step actually touched, and a traceability table from acceptance
criteria to steps to real code paths. Two consequences that are not optional:

- **Keep it current in the same change set as the code.** A new session must be able to resume
  from the plan alone.
- **Keep traceability honest.** When a requirement changes, the chain spec → plan → code is how
  anyone sees which code the change reaches.

## Commands

These `/sdd-*` commands drive the workflow. Each one is a single body file under
`.claude/commands/` that Claude Code runs directly and GitHub Copilot reaches through a
one-line loader in `.github/prompts/`. Neither entry point is advertised to the model: the
workflows run only when you invoke them.

| Command | Purpose |
| --- | --- |
| `/sdd-overview` | Workflow overview, current spec status, command list |
| `/sdd-setup` | Onboarding wizard: `DocLanguage`, seed Memory Bank, first architecture snapshot |
| `/sdd-specify` | Adaptive product-owner interview → lean spec with testable acceptance criteria |
| `/sdd-plan` | Spec → persisted baby-step plan file (research, resume, impact analysis) |
| `/sdd-compile` | Readiness check: tests, acceptance criteria, docs sync |
| `/sdd-architecture-update` | Detect drift, update snapshot + Memory Bank (confirmation gate) |
| `/sdd-lifecycle` | Spec status and moves between backlog/active/done |
| `/sdd-style-update` | Capture coding style preferences into `AGENTS.md` |
