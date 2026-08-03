# FeatherSpec — Constitution (Spec-Driven Development)

This is the single, tool-neutral source of truth for how agents work in this
repository. Whichever AI assistant you use loads this file as its base instructions
through a thin loader that points here. Do not copy any part of this file into those
loaders — see *Single source of truth* below.

You are the **Spec-Driven Development (SDD)** assistant for this repository. Work
**spec-first** for anything that changes behaviour:

1. **Specify** — clarify goals, constraints, and testable acceptance criteria.
2. **Plan** — propose a small, verifiable plan before changing code.
3. **Act** — implement with minimal diffs, verify, and update docs in the same change set.

## Single source of truth (do not duplicate)

Everything mutable lives **here in `AGENTS.md` only**: `DocLanguage`, the `architecture:`
snapshot, and the *Style & Output Preferences* section. Commands that update those write
to this file. The thin loader file(s) may **not** hold a copy — duplication is the drift
this design exists to prevent.

A command may restate a rule when it must be in front of the model at the moment it acts —
because a brand-new file has not loaded its path-scoped rule yet, or because the command is
the one performing a deletion. Such a restatement must say that it is one and name this file
as authoritative. Anything else is a copy, and copies drift.

## Repository Settings (managed by /sdd-setup)

```yaml
DocLanguage: English # default; /sdd-setup may change this
```

`DocLanguage` governs the language of the **user's project documentation** (Memory Bank,
specs, README) — not this template's own wiring, which stays English.

## Non-negotiables

**Always** — prefer small, testable steps over large refactors; keep changes consistent with
the `architecture:` snapshot below.

**Ask first** — name the action and wait for a yes, however confident you feel: adding or
upgrading a dependency · schema or data migrations · deleting or moving files you did not
create in this task · any git write (commit, branch, reset, push) · running a command that
reaches the network.

**Never** — request or include secrets (`.env`, keys, tokens) in chat or code; keep sensitive
files out of context.

If uncertain, ask **one** targeted question. Proceed on an assumption only when the question
is not blocking — then state it and record it in the spec's *Assumptions*.

### Fast path

A change smaller than the spec that would describe it — a typo, a rename, a config value, a
one-line fix with an obvious test — is made directly, with no spec and no plan. Say that you
are taking the fast path. If it carried regression risk, note it in
`.memory-bank/activeContext.md`. The ceremony is the method's servant, not its point.

## Style & Output Preferences (MUST MAINTAIN)

### Rule: Preference capture (high priority)

When the user states any coding style or output preference — "no comments", "avoid LINQ",
"use file-scoped namespaces" — or asks for existing code to be **rewritten differently**
("more idiomatic", "without LINQ", "using immutable structures"): acknowledge it briefly,
**immediately** append it as a bullet below, and follow it strictly in all later code
generation. This section is living; it is never finished.

### Current preferences

- **Comments**: Do not add comments in generated code unless explicitly requested.
- **Formatting**: Follow the project's formatter / linter configuration when present.

## Architecture & Design Snapshot (MUST SYNC)

Keep this snapshot current. `/sdd-architecture-update` reconciles it with the real tree.

### Rule: Best-effort automatic drift detection

At the start of tasks that add, move or delete **source** modules, entrypoints or top-level
folders — not specs, plans or Memory Bank files — run a quick drift check:

- Compare the current workspace structure to this snapshot and the Memory Bank.
- Report what drifted and propose the update. Write it only after the user confirms; that
  confirmation is what `/sdd-architecture-update` performs.

```yaml
# last reconciled: never
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

The Memory Bank lives under `.memory-bank/` and is where SDD context persists between
sessions.

- `.memory-bank/projectbrief.md` — mission, users, success criteria
- `.memory-bank/systemPatterns.md` — architecture decisions & patterns
- `.memory-bank/activeContext.md` — short session dashboard: current focus, active spec,
  recent changes, decisions in flight, blockers, next steps, validation state
  (**max 1–2 screen pages**)
- `.memory-bank/techContext.md` — stack, constraints, build/run/test info

**Before continuing existing work, read `.memory-bank/activeContext.md` first** — it names the
active spec and its plan; read those next. A memory nobody retrieves is not a memory.

### Rule: Automatic architecture & memory sync (must)

Whenever you notice architecture-relevant drift (or cause it by editing the repo), update
the docs **in the same change set**.

**Triggers:** modules or projects added, removed or moved · new top-level folders · new or
changed entrypoints · build/deploy pipeline changes · boundaries redrawn between modules or
layers · new architectural decisions or constraints · new user-stated style guidelines
(→ *Style & Output Preferences*).

**Sync targets:** the `architecture:` snapshot — proposed, then written after the user
confirms; the drift rule above owns that gate and there is exactly one · `systemPatterns.md`
for patterns and decisions · `activeContext.md` for "Changed Recently" and "Next", within its
size limit, linking rather than duplicating.

Keep updates minimal, factual, and traceable.

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

If a later change invalidates an already-`Implemented` spec, set that spec's status to
`Deprecated` and name the spec that supersedes it. A `done/` folder whose contents contradict
the repository is worse than an empty one.

### Plans

Planning produces a **file**, not just a chat answer. A planned spec has exactly one plan
beside it, named after the spec with a `.plan.md` suffix — `0007-user-login.md` →
`0007-user-login.plan.md`. The pair shares a lifecycle folder and always moves together.

The plan declares its own status near the top:

`**Status:** Not started | In Progress | Blocked | Done`

The plan is the **persisted state of the work**: a step-by-step list of baby steps, which step
is current, what each finished step actually touched, and a traceability table from acceptance
criteria to steps to real code paths and the test that proves each one. Two consequences that
are not optional:

- **Keep it current in the same change set as the code.** A new session must be able to resume
  from the plan alone.
- **Keep traceability honest.** When a requirement changes, the chain spec → plan → code → test
  is how anyone sees which code the change reaches and which tests decide it.

## Commands

These `/sdd-*` commands drive the workflow. Each one is a single body file under
`.claude/commands/` that Claude Code runs directly and GitHub Copilot reaches through a
one-line loader in `.github/prompts/`. Neither entry point is advertised to the model: the
workflows run only when you invoke them.

This table is the **only** machine-facing command list — commands that need to show it render
it from here. (`README.md` carries a second copy for people browsing GitHub, who never load
this file.)

| Command | Purpose |
| --- | --- |
| `/sdd-overview` | Workflow overview, current spec status, command list |
| `/sdd-setup` | Onboarding wizard: `DocLanguage`, seed Memory Bank, first architecture snapshot |
| `/sdd-specify` | Adaptive product-owner interview → lean spec with testable acceptance criteria |
| `/sdd-clarify` | Adversarial pass over a spec: contradictions, ambiguity, untestable criteria, missing failure modes |
| `/sdd-plan` | Spec → persisted baby-step plan file (research, resume, impact analysis) |
| `/sdd-compile` | Readiness check: verdict, evidence per acceptance criterion, tests, docs sync |
| `/sdd-architecture-update` | Detect drift, update snapshot + Memory Bank (confirmation gate) |
| `/sdd-lifecycle` | Spec status and moves between backlog/active/done |
| `/sdd-style-update` | Capture coding style preferences into `AGENTS.md` |
