---
description: Onboarding wizard — DocLanguage, Memory Bank, architecture snapshot, working agreements (quality gates, TDD cadence).
argument-hint: "[docLanguage] [projectName] [stack] — or just answer the wizard"
disable-model-invocation: true
---

<!-- Single source for the /sdd-setup workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-setup.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-setup — SDD Setup Wizard

The user may name a documentation language, project name, or stack after the command. If any
are missing, ask for them in the wizard below.

You are running an onboarding wizard for this repository.

## Spec-Driven Development (SDD) — orientation for the model

On the first invocation, start with a short introduction to Spec-Driven Development and the
available `/sdd-*` commands. Then immediately ask **Step 0** (the language question).

Use this definition verbatim (unchanged, in English), optionally with a brief surrounding
explanation:

**Spec-Driven Development (SDD)** is an AI-assisted development approach where a **clear,
structured, versioned specification** is the **primary artifact**. Implementation, tests
(and often documentation) are **derived from the spec** and then **continuously validated
against it** in an iterative loop (**generate → verify/validate → refine**).

For quality with coding assistants: because assistants/agents generate code **anchored to
an explicit spec**—and outputs can be **checked, corrected, and regenerated** based on that
spec—SDD helps **ensure higher-quality, more maintainable code** than unstructured prompt-
or "vibe"-driven coding.

Key principles:

- **Spec-first:** define _what_ to build (behavior, acceptance criteria, constraints) before writing code.
- **Structured specification:** structured enough for consistent tool/agent execution.
- **Living spec:** the spec evolves with the system and remains the reference point for change.
- **Spec-anchored:** the spec survives implementation and is maintained alongside the code it
  produced, so a change to a requirement can be traced to the code and tests it reaches. The
  spec steers the code; it does not replace it, and nothing here regenerates code from a spec.

Explain in 2–4 English sentences that specs are the primary reference point and that the
`/sdd-*` commands guide the workflow. State this repo's operating protocol exactly as
`AGENTS.md` defines it at the top — three steps, never paraphrased into your own words.

## Overview: when to use which command

Show the `/sdd-*` commands from the **Commands** table in `AGENTS.md` — all of them, in
`DocLanguage`. That table is the single machine-facing roster; keep no second copy here, or the
one command a new user never hears about will be the one you forgot to list.

Name the usual order in one line, rendered from the Flow line under *Commands* in `AGENTS.md`.
Keep this overview short, then transition into the onboarding dialog, starting with Step 0.

## Step 0 (MUST be the first question)

Ask exactly: **"In which language should the project documentation Markdown files be written?"**

- If the user does not answer, infer it from the conversation language.
- After a language is chosen, set `DocLanguage` in `AGENTS.md` (the **only** place it lives).
- Then write all project docs (Memory Bank + specs) in that language, rewriting the default
  English templates if needed.
- From here on, conduct the entire dialogue — every question, confirmation and summary —
  in `DocLanguage`. Only this template's own files stay English.

## Step 1 (MUST follow Step 0): Project mode

Run `git ls-files` and count source files outside this template's own folders
(`.claude/`, `.github/`, `.specs/`, `.memory-bank/`, `.vscode/`). Guess the mode —
substantial source files plus a snapshot still at `TBD` or `# last reconciled: never`
suggests existing software — state your guess, then ask exactly this question,
rendered in `DocLanguage`:
**"Is this a new project, or existing software we are adopting?"**

- **New project** → continue with this wizard unchanged.
- **Existing software** → present the choice as a short plain-language briefing in
  `DocLanguage` (~5 lines), and **recommend the deep architecture scan**: it builds
  the architecture fingerprint from the code, and with it agents jump straight to the
  right files instead of searching — cheaper and faster in every later session, higher
  quality, fewer wrong assumptions, and technical planning starts from a shared map.
  Say honestly that the scan reads code and, depending on project size, takes time and
  noticeable tokens. Name the alternatives: describe the architecture yourself, or
  seed only the Memory Bank now and scan later. If the scan is chosen: first collect
  wizard steps 1, 2, 7 and 8 (only the human knows mission, audience, working mode and
  taste), then run the `/sdd-architecture-scan` workflow yourself, exactly as if the user
  had typed it (its body lives in `.claude/commands/sdd-architecture-scan.md`). Skip wizard
  steps 3–5 — the scan answers them from the code and writes `techContext.md` and
  `systemPatterns.md` itself; step 6 shrinks to one confirmation question over the gates
  the scan found. Afterwards finish only actions B (projectbrief + activeContext, plus the
  *Quality gates* section in `techContext.md` from the step-6 confirmation), D, E and F.

## Read the repo before asking

Manifests and lock files (stack, package manager) · the tree's top two levels (entrypoints,
modules) · CI config and test scripts (quality gates) · any README · the existing
`.memory-bank/*` files and the current `AGENTS.md` (snapshot, style preferences). Never ask
for what you can read: pre-fill steps 3, 5 and 6 below from what you found and present them
for correction in one turn. On a re-run, merge with what exists — never reset a curated file.

## Wizard steps (ask only what the repo did not answer)

Every question carries one short clause of why it is asked (what it seeds) — plain
language, no lecture.

1. **Project name & one-liner**
2. **Primary users / target audience**
3. **Tech stack** (languages, frameworks, runtime, package managers) — pre-fill from manifests
4. **Architecture style** (modular monolith / microservices / layered / hexagonal / other) —
   this one needs interpretation, so ask even when you have a guess, and say what you guessed
5. **Repo entrypoints** (apps, services, CLIs, APIs) — pre-fill from the tree
6. **Definition of Green** (the post-implementation quality gate) — pre-fill a proposal from
   the stack and CI config, then have the user confirm or correct it. Propose the command
   sequence that must pass with **zero warnings and zero errors** after every completed
   implementation step, as exact commands with flags, not tool names — e.g. JS/TS:
   `npx eslint .` → `npx tsc --noEmit` → `npm test`; .NET:
   `dotnet format --verify-no-changes` → `dotnet build -warnaserror` → `dotnet test`;
   Python: `ruff check` → `mypy` → `pytest`. Name the loop plainly: on any warning or
   error, fix the code and re-run the gate until it is clean.
7. **Working mode (TDD cadence)** — which cadence binds implementation steps? One short
   plain-language line per option: **(a) strict slice gate** — one test per slice, red first
   via a `Not implemented` stub at the new boundary, stop for the user's go before making it
   green (recommend for teams reviewing every step); **(b) red-first** without per-slice
   stops; **(c) tests alongside implementation**. In every mode, a new test that is green
   immediately because existing code already covers it is fine.
8. **Coding preferences** (comments, naming, patterns to avoid)

## Actions you must perform

After collecting answers:

**A) Ensure folders** (already present in this template; create only if missing):
`.memory-bank/`, `.specs/backlog/`, `.specs/active/`, `.specs/done/`, `.specs/plan-archive/`.

**B) Initialize documentation** in `DocLanguage`:

- Update `.memory-bank/projectbrief.md` with mission + primary users + success criteria.
- Update `.memory-bank/techContext.md` with stack + build/run/test, plus a *Quality gates*
  section listing the confirmed Definition-of-Green commands in order — `/sdd-plan` reads
  them from here, and `/sdd-compile` re-runs them via the plan's *Quality gates* line.
- Create `.memory-bank/activeContext.md` only if missing or still placeholder (`TBD`) —
  otherwise leave it, it may hold live session state. **Read `.claude/rules/memory-bank.md`
  first** and follow its *Structure* section exactly — that section is the only definition of
  this file's shape, and a brand-new file does not load the rule on its own.
- Create/update `.memory-bank/systemPatterns.md` with initial patterns/decisions.

**C) Initial architecture capture (best-effort):**

- Inspect the current workspace structure.
- Populate the `architecture:` snapshot in `AGENTS.md` from the folder/project layout.
- If assumptions are required, write them down and ask the user to confirm (one question).

**D) Style & working preference capture:**

- Seed *Style & Output Preferences* in `AGENTS.md` with what the user stated — in English
  (`AGENTS.md` is wiring), one bullet per preference. Two bullets come from the wizard:
  **Quality gate (code)** — run the Definition-of-Green commands from `techContext.md` after
  every completed implementation step and loop until zero warnings and errors, **scoped
  explicitly: it binds implementation steps only, never specify/clarify/plan work or
  doc-only edits** — and **TDD** with the cadence chosen in step 7.

**E) Budget check:**

- Measure `AGENTS.md` against the cap in `.claude/rules/constitution.md`; if clearly over,
  beyond its tolerance clause, propose one eviction per its order before finishing.

**F) Baseline commit (Ask-first):**

- If the repository has no commit yet, propose one now (e.g. `chore: adopt FeatherSpec and
  seed project docs`). A first commit anchors plan baselines, scope checks and safe
  lifecycle moves (`git mv`); without it every later safety net runs blind. The git write
  stays behind the Ask-first gate in `AGENTS.md`.

## Output

- Summarize what you created/updated and what you inferred.
- Suggest 1–3 next commands: `/sdd-specify`, `/sdd-plan`, `/sdd-architecture-update`, `/sdd-lifecycle`.
