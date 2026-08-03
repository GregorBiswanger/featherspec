---
name: sdd-setup
description: Onboarding wizard — set DocLanguage, seed the Memory Bank, capture the first architecture snapshot.
argument-hint: "[docLanguage] [projectName] [stack] — or just answer the wizard"
disable-model-invocation: true
---

<!-- Shared skill for Claude Code and GitHub Copilot. Deliberately no shell injection
     and no argument-variable substitution: Copilot supports neither, and this one file
     is read by both tools. -->

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
- **Spec-as-source:** the spec is the enduring source of truth; code is (re)generatable from it.

Explain in 2–4 English sentences that specs are the primary reference point, that work here
follows "specify → plan → implement → validate against the spec", and that the `/sdd-*`
commands guide exactly this workflow.

## Overview: when to use which command

- `/sdd-setup`: one-time onboarding — sets `DocLanguage`, prepares the Memory Bank and spec folders, initializes the first architecture snapshot.
- `/sdd-specify`: adaptive product-owner interview that turns a feature/idea into a lean, testable spec under `.specs/`.
- `/sdd-plan`: produce a step-by-step implementation plan for an existing spec.
- `/sdd-architecture-update`: reconcile the architecture snapshot with the real repo after structural change (with confirmation).
- `/sdd-lifecycle`: maintain spec status and move specs between `backlog/`, `active/`, `done/`.
- `/sdd-style-update`: capture or refine coding-style preferences in `AGENTS.md`.

Keep this overview short. Then transition into the onboarding dialog, starting with Step 0.

## Step 0 (MUST be the first question)

Ask exactly: **"In which language should the project documentation Markdown files be written?"**

- If the user does not answer, infer it from the conversation language.
- After a language is chosen, set `DocLanguage` in `AGENTS.md` (the **only** place it lives).
- Then write all project docs (Memory Bank + specs) in that language, rewriting the default
  English templates if needed.

## Wizard steps (ask in order; friendly & minimal)

1. **Project name & one-liner**
2. **Primary users / target audience**
3. **Tech stack** (languages, frameworks, runtime, package managers)
4. **Architecture style** (modular monolith / microservices / layered / hexagonal / other)
5. **Repo entrypoints** (apps, services, CLIs, APIs)
6. **Quality gates** (tests, lint/format, CI)
7. **Coding preferences** (comments, naming, patterns to avoid)

## Actions you must perform

After collecting answers:

**A) Ensure folders** (already present in this template; create only if missing):
`.memory-bank/`, `.specs/backlog/`, `.specs/active/`, `.specs/done/`.

**B) Initialize documentation** in `DocLanguage`:

- Update `.memory-bank/projectbrief.md` with mission + success criteria.
- Update `.memory-bank/techContext.md` with stack + build/run/test.
- Create `.memory-bank/activeContext.md` with this compact structure: metadata lines
  `Last updated: <date>`, `Current branch: <branch>`, `Current phase: <phase>`, then
  sections `## Now`, `## Active Spec`, `## Changed Recently`, `## Decisions in Flight`,
  `## Blockers / Questions`, `## Next`, `## Validation`. Max 1–2 screen pages; do not
  duplicate content from other Memory Bank files.
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
