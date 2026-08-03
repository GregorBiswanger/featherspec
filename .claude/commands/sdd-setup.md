---
description: Onboarding wizard — set DocLanguage, seed the Memory Bank, capture the first architecture snapshot.
argument-hint: "[docLanguage] [projectName] [stack] — or just answer the wizard"
disable-model-invocation: true
---

<!-- Single source for the /sdd-setup workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the one-line loader in
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

Explain in 2–4 English sentences that specs are the primary reference point, that work here
follows "specify → plan → implement → validate against the spec", and that the `/sdd-*`
commands guide exactly this workflow.

## Overview: when to use which command

Show the `/sdd-*` commands from the **Commands** table in `AGENTS.md` — all of them, in
`DocLanguage`. That table is the single machine-facing roster; keep no second copy here, or the
one command a new user never hears about will be the one you forgot to list.

Name the usual order in one line: `/sdd-specify` → `/sdd-clarify` → `/sdd-plan` → read the plan
→ implement → `/sdd-compile` → `/sdd-lifecycle`. Keep this overview short, then transition into
the onboarding dialog, starting with Step 0.

## Step 0 (MUST be the first question)

Ask exactly: **"In which language should the project documentation Markdown files be written?"**

- If the user does not answer, infer it from the conversation language.
- After a language is chosen, set `DocLanguage` in `AGENTS.md` (the **only** place it lives).
- Then write all project docs (Memory Bank + specs) in that language, rewriting the default
  English templates if needed.

## Read the repo before asking

Manifests and lock files (stack, package manager) · the tree's top two levels (entrypoints,
modules) · CI config and test scripts (quality gates) · any README. Never ask for what you can
read: pre-fill steps 3, 5 and 6 below from what you found and present them for correction in
one turn, rather than asking them cold.

## Wizard steps (ask only what the repo did not answer)

1. **Project name & one-liner**
2. **Primary users / target audience**
3. **Tech stack** (languages, frameworks, runtime, package managers) — pre-fill from manifests
4. **Architecture style** (modular monolith / microservices / layered / hexagonal / other) —
   this one needs interpretation, so ask even when you have a guess, and say what you guessed
5. **Repo entrypoints** (apps, services, CLIs, APIs) — pre-fill from the tree
6. **Quality gates** (tests, lint/format, CI) — pre-fill from CI config and test scripts
7. **Coding preferences** (comments, naming, patterns to avoid)

## Actions you must perform

After collecting answers:

**A) Ensure folders** (already present in this template; create only if missing):
`.memory-bank/`, `.specs/backlog/`, `.specs/active/`, `.specs/done/`.

**B) Initialize documentation** in `DocLanguage`:

- Update `.memory-bank/projectbrief.md` with mission + success criteria.
- Update `.memory-bank/techContext.md` with stack + build/run/test.
- Create `.memory-bank/activeContext.md`. **Read `.claude/rules/memory-bank.md` first** and
  follow its *Structure* section exactly — that section is the only definition of this file's
  shape, and a brand-new file does not load the rule on its own.
- Create/update `.memory-bank/systemPatterns.md` with initial patterns/decisions.

**C) Initial architecture capture (best-effort):**

- Inspect the current workspace structure.
- Populate the `architecture:` snapshot in `AGENTS.md` from the folder/project layout.
- If assumptions are required, write them down and ask the user to confirm (one question).

**D) Style preference capture:**

- Seed *Style & Output Preferences* in `AGENTS.md` with what the user stated.

## Output

- Summarize what you created/updated and what you inferred.
- Suggest 1–3 next commands: `/sdd-specify`, `/sdd-plan`, `/sdd-architecture-update`, `/sdd-lifecycle`.
