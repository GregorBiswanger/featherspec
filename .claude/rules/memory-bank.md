---
paths:
  - .memory-bank/**/*.md
---

# Memory Bank writing rules

- Write in the language specified by `DocLanguage` in `AGENTS.md`.
- Keep entries concise, factual, and actionable.
- If architecture changes, update `systemPatterns.md` and record "Changed Recently" in `activeContext.md`.

## Rules for `activeContext.md`

`activeContext.md` is a **short, current working and handoff context** for the agent — not a
changelog, not a wiki, and not a second spec. It must answer in 30 seconds: *what are we
working on, which spec is authoritative, what was recently decided/changed, what is next, and
what does the agent need to know right now?*

**Keep it to max 1–2 screen pages.** If it grows longer, information belongs in a more
permanent file.

### Structure

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
3. After a phase transition: Requirements → Design → Tasks → Implementation → Validation.
4. Before a context reset or new agent session.
5. After a bugfix with regression risk.
6. **Always** when switching from one active spec to another.

### What does NOT belong here

Move the following to the correct file instead of adding it to `activeContext.md`:

| Content | Correct file |
|---|---|
| Full requirements | spec `requirements.md` |
| Full technical design | spec `design.md` or `systemPatterns.md` |
| Full task list | spec `tasks.md` |
| Long-term architecture decisions | `systemPatterns.md` |
| Tech stack, setup, dependencies | `techContext.md` |
| Project vision, goals, scope | `projectbrief.md` |
| Historical progress over weeks | a `progress.md` |
| Stale todos | delete or move to `progress.md` |

**Do not duplicate. Link and summarize.**
