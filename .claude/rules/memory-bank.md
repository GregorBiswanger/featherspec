---
paths:
  - .memory-bank/**/*.md
---

# Memory Bank writing rules

The Memory Bank file list, each file's purpose, and the size limit on `activeContext.md` are
defined in `AGENTS.md` (loaded every session). These rules cover only what is specific to
**writing Memory Bank files**:

- Write in the language specified by `DocLanguage` in `AGENTS.md`.
- Keep entries concise, factual, and actionable.
- If architecture changes, update `systemPatterns.md` and record "Changed Recently" in `activeContext.md`.

> Note: path-scoped rules load when a matching file is **read**. A brand-new Memory Bank file
> has not been read yet, so this rule would not be loaded when `/sdd-setup` creates one — which
> is why that command is instructed to read this file first rather than carry its own copy of
> the structure. `AGENTS.md` stays authoritative for the file list and the size limit.

## Rules for `activeContext.md`

`activeContext.md` is a **short, current working and handoff context** for the agent — not a
changelog, not a wiki, and not a second spec. It must answer in 30 seconds: *what are we
working on, which spec is authoritative, what was recently decided/changed, what is next, and
what does the agent need to know right now?*

If it outgrows the size limit stated in `AGENTS.md`, the surplus belongs in a more permanent
file — see the table below.

### Structure

This is the **only** definition of the file's shape; commands that create it point here.

Three metadata lines first — `Last updated: <date>`, `Current branch: <branch>`,
`Current phase: specify | plan | act` — then these sections:

- **Now** — one sentence: current goal of this session/work track.
- **Active Spec** — link to the relevant spec file(s), current task ID, acceptance criteria in focus.
- **Changed Recently** — only actual changes since the last update: files/modules affected, tests/checks run.
- **Decisions in Flight** — short-lived decisions not yet permanent enough for `systemPatterns.md`.
- **Blockers / Questions** — open items blocking progress.
- **Next** — numbered, concrete, immediately actionable steps.
- **Validation** — current test/check status: done, pending, known issues.

### When to update

1. At the end of each relevant coding session.
2. After an important decision.
3. After a phase transition (specify → plan → act) or a lifecycle move (backlog → active → done).
4. Before a context reset or new agent session.
5. After a bugfix with regression risk.
6. **Always** when switching from one active spec to another.

**Refresh the `Last updated` line on every one of these.** A date that was set once and never
touched again is worse than no date: it makes a stale file look current.

### What does NOT belong here

Move the following to the correct file instead of adding it to `activeContext.md`:

| Content | Correct file |
|---|---|
| Full requirements | the spec under `.specs/` |
| Full technical design | the spec, or `systemPatterns.md` if it is long-lived |
| Full step list | the spec's `.plan.md` sibling |
| Long-term architecture decisions | `systemPatterns.md` |
| Tech stack, setup, dependencies | `techContext.md` |
| Project vision, goals, scope | `projectbrief.md` |
| Historical progress over weeks | nowhere — that is what git history is for |
| Stale todos | delete them, or move the live ones to the plan's `Notes:` |

**Do not duplicate. Link and summarize.** Never route content into a file this template does
not declare: the Memory Bank is these four files, and a spec's only companion is its `.plan.md`.
