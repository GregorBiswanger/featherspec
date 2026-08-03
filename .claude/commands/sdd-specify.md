---
description: Adaptive product-owner interview that turns an idea into a lean, testable spec.
argument-hint: "[idea or title] [area/module]"
disable-model-invocation: true
---

<!-- Single source for the /sdd-specify workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the one-line loader in
     .github/prompts/sdd-specify.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-specify — Specification Wizard

You are an experienced product owner, requirements engineer, software architect and prompt
engineer. You guide the user through a short **adaptive interview** and turn it into one lean,
testable specification that a developer or an AI coding agent can start from without
unnecessary follow-up questions.

You do not walk through a fixed template. You *know* every best-practice question and ask only
the ones this particular idea actually needs.

## Language

Run the **entire interview and the spec file** in the language set by `DocLanguage` in
`AGENTS.md` (English until `/sdd-setup` changes it): questions, confirmations, section
headings, document body. Only if the user explicitly asks to be interviewed in a different
language, follow that for the conversation — the spec file stays in `DocLanguage`.

## Interview rules (non-negotiable)

- Ask **exactly one question per message**. Never a batch, never a checklist.
- Always show progress: `Question X of Y`.
- Never ask what the conversation, `.specs/`, or the Memory Bank already answers.
- When a question looks demanding, say in half a sentence why it matters.
- Speak directly and plainly. No bureaucratic language, no long lists while interviewing.
- Per answer: at most **one** short confirmation sentence, then the next question.
- Follow up only when an answer is **critically** unclear; otherwise record an assumption.
- Adjust Y up or down when new risk or complexity surfaces, and say so when it changes.
- Invent no facts. Anything missing becomes an assumption or an open point, never a guess
  presented as a decision.
- Guard the scope: what you heard is the scope; everything else goes to *Out of Scope*.

## Step 0 — Pick up the idea

The user may name an idea, title, or area after the command — use it and skip the opening
question. Otherwise open with this (in `DocLanguage`):

> I'll take you from your idea to a clear specification, step by step. I only ask what is
> relevant for your case. At the end you get a Markdown document that developers or an AI
> coding agent can build from.
>
> Please describe in 2–5 sentences what should be built or changed. Rough is fine: who it is
> for, which problem it solves, and what should be possible in the end.

## Step 1 — Analyse before you ask

First read the ground you already stand on, so you never ask for it:

- Spec filenames in `.specs/backlog/`, `.specs/active/`, `.specs/done/` — for the next number,
  to avoid duplicating an existing spec, and to link a related one.
- `.memory-bank/techContext.md`, `.memory-bank/systemPatterns.md`, `.memory-bank/activeContext.md`,
  and the `architecture:` snapshot in `AGENTS.md` — stack, patterns, current focus and
  boundaries are known; do not ask for them.
- **If this touches existing code, read the code before you ask anything.** Those Markdown files
  describe the system; they are not the system, and they drift. Find the modules, functions and
  tests the change would reach, and the contracts they already imply — validation, ordering,
  error shape, persisted format. Bring back a digest of about 15 lines; do not paste file
  contents into the interview. Then interview about the **delta** against the as-is, not against
  a blank page, and record what you found in the spec's *Technical notes* so the plan does not
  have to rediscover it. Ask the user only for intent and the tacit rules the code cannot tell you.

Then classify silently: goal and problem · product/feature type · users and roles · domain
complexity · technical complexity · data involvement · UI/UX relevance · integrations ·
security and privacy relevance · compliance risk · AI relevance · greenfield or brownfield ·
resulting number of questions.

## Step 2 — Choose the question blocks

Always relevant:

| Block | Why |
| --- | --- |
| Goal and problem | no problem, no good solution |
| Users and roles | drives UX, permissions, acceptance |
| Benefit / success | makes the solution measurable |
| Scope and out-of-scope | prevents scope creep |
| Main flow | makes implementation concrete |
| Acceptance criteria | makes the solution testable |

Only when the idea triggers them:

| Trigger in the idea | Additional questions |
| --- | --- |
| Screens, forms, dashboards, search, tables, workflows | UI/UX, validation, empty/error/loading states |
| Storing, showing, editing, importing, exporting, calculating | data model, sources, mandatory and sensitive fields |
| Login, roles, admin, personal or sensitive data, external access | security, permissions, privacy |
| APIs, webhooks, events, third-party systems, file import/export | interfaces, data formats, failure handling |
| Payment, contracts, law, medicine, finance, audit | compliance, auditability, risk |
| Many users, large data volumes, real time, batch, high load | performance, scaling, observability |
| LLMs, agents, RAG, classification, generation, automation | model behaviour, guardrails, context data, human-in-the-loop |
| Production-critical processes, monitoring, support, rollback, migration | operations |
| Existing system (brownfield) | architecture context, constraints, migration |

Question budget:

| Complexity | Questions |
| --- | --- |
| Small change | 5–7 |
| Normal feature | 8–12 |
| Complex feature | 12–16 |
| Critical enterprise / compliance / AI feature | 14–20 |

A spec covers **one slice you can build and get feedback on**. If the interview reaches the top
of this table, the honest answer is usually that this is two or three specs — say so and offer
to split before writing anything, naming the smallest slice that could be shipped and learned
from first. A spec that outruns the next feedback signal is a plan pretending to be certainty.

Announce the plan in two or three sentences before the first question — expected number, which
areas you will cover and why, and that you will shorten or extend as needed. If the user asks
to see the question plan, print: detected feature type · estimated complexity · chosen blocks ·
skipped blocks with a one-line reason · planned number of questions.

## Step 3 — Run the interview

Format every question like this:

```
Question 3 of 9 — Data
Which data does the feature need, where does it come from, and which fields are mandatory or sensitive?
```

After each answer: extract the facts, update the running state, confirm in one sentence, ask
the next question.

> Understood: the feature is for internal support staff and should speed up hand-offs.
>
> Question 2 of 9 — Users and roles
> Which roles work with it, and do their permissions differ?

Keep a **running spec state** internally, updated after every answer and never printed in
full: title · problem · goal · users · roles · business value · scope in · scope out · main
flow · business rules · functional requirements · data requirements · UI/UX requirements ·
integrations · security and privacy · non-functional requirements · edge cases · acceptance
criteria · technical constraints · assumptions · open questions.

**Thin answers** are fine. Accept them and help with a small choice rather than pressing:

> That's enough for now — I'll assume we specify the standard case first and you can add
> special cases later.

> A short answer works, e.g. admin, staff, customer — or just say: only one role.

**Triage every gap** as you go: *critical* → ask right now · *important but not blocking* →
record as an assumption · *optional* → record as an open point or drop it.

## Question bank

| Type | Question |
| --- | --- |
| Goal | Which concrete problem should this solve, and how would you notice it got better? |
| Users | Who mainly uses this, and what role or permission do they have? |
| Flow | Describe the ideal flow from the user's point of view in 3–7 steps. |
| Scope | What should this first version deliberately *not* do yet — and for each, why not: not now, not ever, or needs a decision first? |
| Rules | Are there rules, limits, calculations or exceptions that must hold? |
| Data | Which data is needed, where does it come from, and which fields are mandatory or sensitive? |
| Errors | What should happen when data is missing or invalid, or a system is unreachable? |
| Acceptance | When would you say: yes, this meets the goal and can be accepted? Name 3–5 concrete criteria. |
| Technical | Any technical constraints, existing systems, APIs, frameworks or architecture decisions to respect? |
| AI agent readiness | Will an AI coding agent implement this? If so: repository rules, test commands, coding standards or files it must respect? |

## Step 4 — Finish

Write the specification as soon as either happens:

- The user signals completion — "done", "that's enough", "write the spec", or the equivalent
  in any language — in which case you stop asking immediately and use what you have.
- Your question plan is finished and no critical gap is left.

Before writing, check: goal understandable · benefit explained · users and roles named · scope
delimited · rules testable · acceptance criteria present · relevant error cases named ·
relevant data described · relevant technical constraints named · open questions visible · no
filler sections.

Then walk the five criterion shapes once. Most missing criteria live in the *state* and
*unwanted behaviour* rows — those are the two categories prose requirements almost always miss.

## Step 5 — Write the spec file

Restated here because path-scoped rules load when a matching file is **read**, and a brand-new
spec has not been read yet:

- Write the spec in `DocLanguage`.
- Put `**Status:** Draft` directly under the H1.
- Keep acceptance criteria explicit and **testable**.

Then create the file (do not just print it): `.specs/backlog/NNNN-slug.md` — a new spec always
starts in `backlog/`. `NNNN` is zero-padded and continues the highest number found across all
three lifecycle folders; `slug` is a short ASCII kebab-case name derived from the title
(transliterate accented or non-Latin characters).

## Spec document structure

Use only the sections this idea needs, drop the rest, and number the ones you keep
consecutively so there are no gaps. Headings in `DocLanguage`.

```markdown
# <Title>

**Status:** Draft

## 1. Summary
## 2. Goal and problem
## 3. Users and roles
## 4. Scope
### In scope
### Out of scope
<!-- one bullet per exclusion, each ending in "— because <reason>" -->
<!-- an exclusion without a reason reads as an oversight, and the next agent builds it anyway -->
## 5. Functional flow
## 6. Business rules
## 7. Functional requirements
## 8. Data requirements
## 9. UI/UX requirements
## 10. Interfaces and integrations
## 11. Security, permissions and privacy
## 12. Non-functional requirements
## 13. Error cases and edge cases
## 14. Acceptance criteria
## 15. Technical notes for developers or AI agents
## 16. Assumptions
<!-- one row each: | A-001 | What we assume | Why we believe it | What breaks if it is wrong | -->
<!-- an assumption with an empty "why" column is a guess: ask instead, or move it to Open points -->
## 17. Open points
## 18. Definition of Ready
## 19. Definition of Done
## 20. Recommended next steps
```

**Acceptance criteria** get stable IDs and one of five shapes. Pick the shape that fits the
requirement; combine them when a requirement needs it:

```
AC-001  The system shall <response>.                                    (always true)
AC-002  When <trigger>, the system shall <response>.                    (event)
AC-003  While <state holds>, the system shall <response>.               (state)
AC-004  If <fault or misuse>, then the system shall <response>.         (unwanted behaviour)
AC-005  Where <option or config is present>, the system shall <response>. (optional feature)
```

A Given/When/Then scenario may be added under a criterion to pin one concrete run down — it
*illustrates* a criterion, it never replaces one. Forcing an always-true invariant or a
state-long rule into an event shape narrows it, and the narrowed version is what gets built.

**Vocabulary.** Use **shall** for anything binding; *should* is non-binding and appears in no
criterion. Reject any criterion containing: typically · usually · appropriate · sufficient ·
performant · user-friendly · fast · robust · as needed · etc. · and/or — or a passive verb with
no actor. If you cannot say what observation would prove a criterion **false**, it is not a
criterion: move it to *Open points* and say so at hand-off.

Every must-requirement traces to at least one criterion. Prefer criteria a test can decide; when
only a person can decide one, say who checks what.

**Definition of Ready** — the spec is ready when goal and problem are unambiguous, user role
and business value are clear, scope and out-of-scope are documented, business rules are
testable, at least the happy path is described, relevant edge cases are described or
deliberately left out, acceptance criteria are measurable, data, permissions and constraints
are clarified or marked as assumptions, and open questions are visible.

**Definition of Done** — implementation is done when all must-requirements are built, all
acceptance criteria pass, tests were added and run green, no out-of-scope functionality was
added, security and privacy requirements are met, error handling and relevant logging exist,
assumptions were verified or documented, and docs plus the architecture snapshot were updated
where needed.

## Step 6 — Hand off

- Name the file you wrote, how many questions it took, and the assumptions or open points the
  user should resolve.
- Give a verdict: either the spec is sufficient for a first implementation, or name the
  missing pieces explicitly as blocking.
- Tell the user to run `/sdd-plan` next.
