<div align="center">

<img src="featherspec-logo.png" alt="FeatherSpec" width="320">

**Spec-driven development that works in Claude Code _and_ GitHub Copilot — from the same files.**

No CLI. No install. No dependencies. Markdown and folders.

[![Use this template](https://img.shields.io/badge/Use%20this%20template-2ea44f?style=for-the-badge&logo=github&logoColor=white)](https://github.com/GregorBiswanger/featherspec/generate)
[![Read the Wiki](https://img.shields.io/badge/Read%20the-Wiki-0b7cbb?style=for-the-badge&logo=readthedocs&logoColor=white)](https://github.com/GregorBiswanger/featherspec/wiki)
[![License: MIT](https://img.shields.io/badge/License-MIT-555?style=for-the-badge)](LICENSE)

</div>

---

A chat forgets. FeatherSpec doesn't.

You describe **what** you want, your agent turns it into a **spec** with testable acceptance
criteria, then into a **plan** of baby steps — and only then writes code. Spec, plan, decisions
and progress all live on disk as Markdown in your repository, so the next session, the next
teammate, and the next tool pick up exactly where you left off.

Eight `/sdd-*` commands drive that loop, and they behave identically in Claude Code and in
GitHub Copilot, because both tools execute the **same files**.

---

## Start in two minutes

### 1 · Get the template

Click **[Use this template](https://github.com/GregorBiswanger/featherspec/generate)** on GitHub — or clone it:

```bash
git clone https://github.com/GregorBiswanger/featherspec.git my-project
```

The clone brings this repository's commit history along with a remote pointing back here. For
your own project you want neither: delete the `.git` folder (`rm -rf .git`, or
`Remove-Item -Recurse -Force .git` in PowerShell) and run `git init`.

**Have Node.js?** Then `degit` saves you that cleanup — it downloads only the current file
state, so there is no history and no `.git` folder to detach in the first place:

```bash
npx degit GregorBiswanger/featherspec my-project
```

### 2 · Open it in your assistant

**Claude Code**

```bash
cd my-project
claude
```

`CLAUDE.md` imports `AGENTS.md` at session start, so the rules are always loaded. Type `/` and
the `sdd-` commands are there.

**GitHub Copilot (VS Code)**

Open **the project folder itself** — not a parent folder, or the slash commands won't show up.
Open Copilot Chat (<kbd>Ctrl</kbd>/<kbd>Cmd</kbd> + <kbd>Alt</kbd> + <kbd>I</kbd>), switch to
**Agent** mode, and optionally pick the **SpecDrivenAgent** persona. Type `/` — same commands,
same behaviour.

### 3 · Run the wizard once

```text
/sdd-setup
```

It asks which language your documentation should be written in (answer `English`, `Deutsch`,
`Français`, … — everything the workflow writes from then on follows it), then a handful of
questions about the project. It seeds the Memory Bank and captures a first architecture
snapshot.

That is the entire installation. Nothing to build, nothing to run.

---

## The loop

```mermaid
flowchart LR
  S["💬 /sdd-specify<br/>interview → spec"] --> P["🗺️ /sdd-plan<br/>spec → baby steps"]
  P --> I["⚙️ implement<br/>step by step"]
  I --> C["✅ /sdd-compile<br/>readiness check"]
  C --> L["📦 /sdd-lifecycle<br/>spec → done/"]
  L -.->|next iteration| S
```

The spec says **what and why**. The plan says **how**, in steps small enough to verify one at a
time. Both are files, both are versioned, and a traceability table connects every acceptance
criterion to the steps and the code paths that fulfil it.

---

## See it work — a 5-minute example

A tiny service that splits a restaurant bill. No database, no frontend, no login — just enough
to watch one full SDD cycle go by.

### ① Say what you want — not how

```text
/sdd-specify A small service that splits a restaurant bill fairly across several people.
A user gives three things: the bill amount, a tip percentage and the number of people.
The service returns what each person pays in total, and the overall tip amount.
```

Notice what is *not* in there: no "build a REST API", no Express, no endpoint paths. Only the
problem. The agent now interviews you — one question at a time — about users, scope, edge cases
and acceptance criteria, then writes:

```text
.specs/backlog/0001-bill-splitter.md
```

### ② Add the product rules only you can decide

As a follow-up message in the same chat:

```text
Add these rules to the spec:
- 0 or fewer people returns a clear error, never a calculation.
- A negative tip percentage is rejected. 0 percent is allowed.
- A bill amount of 0 is allowed and yields 0 per person.
- Leftover rounding cents go to the first person, so the sum matches the total exactly.
```

That last rule is the point of the whole exercise. It is a **product decision** — no model can
guess it, and no developer should invent it. In the spec it becomes a testable criterion:

```text
AC-004: Given a bill of 100 split across 3 people, when the split is calculated,
        then the three amounts sum to exactly 100.00.
```

### ③ Let the agent plan the how

```text
/sdd-plan Build it as a minimal HTTP service on Node.js with the built-in http module, no
frameworks. One POST endpoint /split. The calculation lives in its own testable module.
Unit tests with the built-in node:test runner. No database, no build step.
```

This writes `0001-bill-splitter.plan.md` right next to the spec: numbered baby steps
(`T-001`, `T-002`, …), each with a `Verify:` line you can actually run, plus a traceability
table and a session-handoff block. Then it **stops** — planning never touches code.

### ④ Implement, step by step

```text
Implement T-001.
```

The agent does one focused change, runs its `Verify:` line, ticks the box in the plan, and
records which files it touched — in the same change set as the code. Repeat until the steps are
done. Close a session mid-way and the next one resumes from the plan file, not from your memory.

### ⑤ Check it against your own criteria

```text
/sdd-compile
```

You get a readiness brief: every acceptance criterion marked satisfied or pending *with
evidence*, open plan steps, whether the docs are in sync, and the next three actions. Tests run
as part of it.

The important discipline: you check against **the criteria you wrote**, not against a gut
feeling. If "actually I'd also like X" comes up now, that is not a bug — it was never in the
spec. That is the next iteration.

### ⑥ Close the loop

```text
/sdd-lifecycle
```

Spec and plan move together into `.specs/done/`, statuses are updated, and the Memory Bank is
refreshed. The spec stays as a living document — the starting point for iteration two, which
runs faster because the context is already written down.

---

## The eight commands

| Command | What it does |
| --- | --- |
| `/sdd-overview` | Where am I? Workflow map, current spec status, command list |
| `/sdd-setup` | One-time wizard: doc language, Memory Bank, first architecture snapshot |
| `/sdd-specify` | Adaptive product-owner interview → a lean, testable spec |
| `/sdd-plan` | Spec → a persisted plan of baby steps, with research and traceability |
| `/sdd-compile` | Readiness check: tests, acceptance criteria, docs sync |
| `/sdd-lifecycle` | Move specs between `backlog/`, `active/`, `done/` |
| `/sdd-architecture-update` | Detect structural drift, update the snapshot (asks first) |
| `/sdd-style-update` | Capture a coding-style preference so it sticks |

New to it? Just run `/sdd-overview`.

---

## What ends up in your repo

```text
AGENTS.md              the constitution — rules, doc language, architecture snapshot
CLAUDE.md              one line: @AGENTS.md

.claude/commands/      the eight workflow bodies (Claude runs them directly)
.claude/rules/         path-scoped craft rules, loaded when a matching file is read
.github/prompts/       one-line loaders so Copilot reaches the same bodies

.specs/                backlog/ · active/ · done/   — specs + their plan files
.memory-bank/          projectbrief · systemPatterns · techContext · activeContext
```

Everything mutable lives in `AGENTS.md` and the two data folders. No rule and no workflow is
written down twice — that is the design, and it is why both tools stay in sync for free.

---

## Why one template for two tools

- **VS Code Copilot reads most of Claude Code's configuration natively** — `AGENTS.md`,
  `.claude/rules/`, hooks in `.claude/settings.json`. FeatherSpec leans on that overlap instead
  of maintaining two copies.
- **Workflows are commands, not skills.** A skill advertises itself to the model on every
  request; a command is only ever run when *you* type it. Eight workflows sitting in every
  system prompt is a cost with no upside here.
- **Only the entry point differs.** `.claude/commands/<name>.md` holds the body;
  `.github/prompts/<name>.prompt.md` is a one-line pointer to it. One file to edit, two tools
  served.

The [full interop matrix](https://github.com/GregorBiswanger/featherspec/wiki/Interop-Matrix)
spells out exactly what each tool reads, with caveats and sources.

---

## Learn more

Everything beyond this page lives in the **[Wiki](https://github.com/GregorBiswanger/featherspec/wiki)**:

| Page | What's in it |
| --- | --- |
| [Getting Started](https://github.com/GregorBiswanger/featherspec/wiki/Getting-Started) | Setup for both tools, verifying what actually loaded |
| [Commands](https://github.com/GregorBiswanger/featherspec/wiki/Commands) | Every `/sdd-*` command in detail |
| [Specs & Plans](https://github.com/GregorBiswanger/featherspec/wiki/Specs-and-Plans) | Document structure, lifecycle, traceability |
| [Memory Bank](https://github.com/GregorBiswanger/featherspec/wiki/Memory-Bank) | The four files and what belongs in each |
| [Interop Matrix](https://github.com/GregorBiswanger/featherspec/wiki/Interop-Matrix) | What Copilot reads from `.claude/`, with sources |
| [Configuration](https://github.com/GregorBiswanger/featherspec/wiki/Configuration) | Shared vs. local settings, auto memory, MCP, hooks |
| [Extending FeatherSpec](https://github.com/GregorBiswanger/featherspec/wiki/Extending-FeatherSpec) | Add your own commands and rules |
| [Hands-On Walkthrough](https://github.com/GregorBiswanger/featherspec/wiki/Hands-On-Walkthrough) | The full workshop exercise, PO/Dev in pairs |
| [Troubleshooting](https://github.com/GregorBiswanger/featherspec/wiki/Troubleshooting) | Commands not showing up, rules not applying |
| [Committing to One Tool](https://github.com/GregorBiswanger/featherspec/wiki/Committing-to-One-Tool) | Strip out the other tool later, mechanically |

Coming from `copilot-spec-driven-template`? See
[Migration](https://github.com/GregorBiswanger/featherspec/wiki/Migrating-from-copilot-spec-driven-template).

---

<div align="center">

Built by [Gregor Biswanger](https://github.com/GregorBiswanger) · MIT licensed · Issues and PRs welcome

</div>
