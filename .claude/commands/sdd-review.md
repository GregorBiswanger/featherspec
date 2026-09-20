---
description: Two-axis review of the work since a fixed point — Standards (repo rules + code smells) and Spec (does the change do what the spec asked?) — reported side by side, never merged.
argument-hint: "[fixed-point (commit/branch/main)] [spec path] — or just answer the questions"
disable-model-invocation: true
---

<!-- Single source for the /sdd-review workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-review.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-review — Two-Axis Review

Review the changes since a fixed point along two **separate axes**, so one cannot mask the other:

- **Standards** — does the change conform to this repo's documented rules (`AGENTS.md`, `.claude/rules/*`, the *Style & Output Preferences*) and to the smell baseline below?
- **Spec** — does the change faithfully implement what the spec asked for: missing or partial requirements, scope creep, requirements that look implemented but are wrong?

A change can pass one axis and fail the other. Report both, never merge or rerank findings across axes.

## Scope vs /sdd-compile

`/sdd-compile` certifies *evidence* — are the acceptance criteria proven? This command reviews *the changes themselves* — quality and spec fidelity of the diff. Run it after implementation, before or alongside `/sdd-compile`; its findings never override the compile verdict and vice versa.

## Step 1 — Pin the fixed point

The user names a fixed point (commit SHA, branch, `main`, `HEAD~N`) or you derive it: the plan's `Baseline:` line if present, else the commit before the first step commit. Ask only if neither exists. In `DocLanguage`.

Validate before anything else:

```
git rev-parse <fixed-point>            # must resolve — a bad ref fails HERE
git log <fixed-point>..HEAD --oneline # the commit list
git diff <fixed-point>...HEAD --stat  # three-dot (merge-base); must be non-empty
```

A bad ref or an empty diff ends the command with a one-line explanation.

## Step 2 — Identify the spec

In this order: the spec path the user gave · the spec named by the plan or `activeContext.md` · an issue/ticket key on the spec's `**Ticket:**` line (fetch it per `.memory-bank/issue-tracker.md`, if `IssueTracker:` is set — body **and comment thread**) · a spec under `.specs/` matching the branch. Nothing found → ask. If there is no spec, the Spec axis reports "no spec available" — never invent criteria.

## Step 3 — Gather the standards

Everything that documents how code should be written here: `AGENTS.md` (Non-negotiables, Style & Output Preferences), `.claude/rules/*` matching the touched paths, a CONTRIBUTING or CODING_STANDARDS file if present. Name what you found; an undocumented repo falls back to the smell baseline alone.

## Step 4 — Run the two axes (separately, in parallel where the tool supports sub-agents)

**Standards axis.** For each file/hunk: (a) violations of a documented rule — cite the rule's file and its wording; (b) smells from the baseline below — name the smell and quote the hunk. Documented-rule breaches can be hard findings; smells are always judgement calls ("possible Feature Envy"), and a documented repo rule overrides the baseline. Skip anything tooling already enforces.

Smell baseline (each reads *what it is* → *how to fix*):

- **Mysterious Name** — a name that doesn't reveal what it does or holds. → rename.
- **Duplicated Code** — the same logic shape in more than one hunk or file. → extract the shared shape.
- **Feature Envy** — a method reaching into another object's data more than its own. → move it onto the data.
- **Data Clumps** — the same few fields or params travelling together. → bundle them into one type.
- **Primitive Obsession** — a primitive standing in for a domain concept. → give the concept its own small type.
- **Repeated Switches** — the same switch/if-cascade recurring across the change. → polymorphism, or one shared map.
- **Shotgun Surgery** — one logical change forcing scattered edits across many files. → gather what changes together.
- **Divergent Change** — one file edited for several unrelated reasons. → split it.
- **Speculative Generality** — abstraction, parameters or hooks for needs the spec doesn't have. → delete it.
- **Message Chains** — long `a.b().c().d()` navigation. → hide the walk behind one method.
- **Middle Man** — a class or function that mostly delegates onward. → cut it.
- **Refused Bequest** — an implementer ignoring most of what it inherits. → composition instead.

**Spec axis.** Against the spec's goals and acceptance criteria: (a) asked-for requirements missing or partial in the changes; (b) behaviour present that the spec did not ask for — scope creep; (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line (or AC id) for every finding.

**Fresh-eyes rule (from /sdd-compile, same rationale):** delegate each axis to a subagent that receives only the diff, the standards/spec text and its brief — no conversation history — whenever the tool supports sub-agents; **mandatory when this session wrote the code**, because the context that produced a gap is the worst placed to find it. Where sub-agents are impossible, say so in one line and run both axes yourself, distrust your own session's notes, and re-read the actual code.

**Reviewer independence:** where a different model is available for review than the one that wrote the changes, use it — a model reviewing its own output inherits its blind spots.

## Step 5 — Report

In `DocLanguage`, under `## Standards` and `## Spec` headings, findings verbatim or lightly cleaned; per finding: severity (hard rule breach / judgement call), file and hunk, what and why, and the fix direction. Then one line per axis: total findings and the worst one. No cross-axis ranking, no single winner.

## Triage (the part sub-agents cannot do)

Findings are hypotheses until verified: read the actual code before accepting one — a reviewer can be right about a symptom and wrong about the fix (e.g. "extract this" breaking a sibling convention the repo endorses). Triage every finding: accept · accept-with-adaptation · reject-with-reason. Record rejected findings with their reason in the plan's or spec's notes if the user wants them kept; never silently drop a finding.

## Output

- The two-axis report.
- A short triage note: which findings you verified against the code and what fell away.
- Suggested next commands: fix and re-run for findings · `/sdd-compile` for the evidence check · `/sdd-style-update` if the review surfaced a preference worth capturing as a bullet.