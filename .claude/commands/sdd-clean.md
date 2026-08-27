---
description: Context cleanup — analyze FeatherSpec's persistent markdown, dedupe and compact safely, with a token report.
argument-hint: "[focus file] (optional)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-clean workflow. Claude Code runs this file directly;
     GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-clean.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-clean — Context Cleanup

Persistent markdown is a paid resource: every line it holds loads into future context
windows. This workflow keeps it at *as little as possible, as much as necessary* — maximum
useful information per token, judged by semantic usefulness, never by raw size alone. It is
manually invokable at any time; other workflows may recommend it, but cleanup never happens
as a silent side effect. The user may name one file after the command; then only that file
is analyzed and compacted. All dialogue and reports in `DocLanguage`; compacted files keep
the language they are written in.

## Scope and write authority

Analyze all FeatherSpec-managed markdown; write only where this command has authority:

| Files | Authority |
| --- | --- |
| `.memory-bank/*.md` | analyze + compact here |
| `.architecture/*.md` maps | analyze + compact here (≤ 40-line budget per `.claude/rules/architecture-map.md`) |
| `architecture:` snapshot in `AGENTS.md` | analyze only; observed drift hands over to `/sdd-architecture-update` — its gate stays the snapshot's only write path |
| `.specs/**` specs and plans | analyze, report findings only — their commands and rules own edits; `AC-` and `T-` IDs are never cleanup material |
| Wiring (`.claude/`, `.github/`, `AGENTS.md` prose) | out of scope — template-owned |

## Phase 1 — Analyze (read-only)

Read each in-scope file against its declared purpose (`AGENTS.md` Memory Bank list,
`.claude/rules/memory-bank.md` with its routing table, `.claude/rules/architecture-map.md`
for maps) and ask, in this order:

1. **Misplaced?** Content in the wrong file moves to its declared home per the routing
   table — moving is not deleting.
2. **Duplicate?** A fact stated in several files keeps exactly one canonical home; the
   others link or drop it.
3. **Stale?** Spot-check claims against the repository — named paths, technologies,
   assumptions. Stale content is updated or removed; git history is the archive, the live
   file represents the current useful state.
4. **Rediscoverable?** Inventories an agent can derive from the code in seconds (file
   lists, obvious signatures) go. Intent, boundaries, constraints, conventions and traps
   stay — the Memory Bank holds knowledge, not a second copy of the repository.
5. **Historical?** `activeContext.md` is the current working state, not a diary: finished
   episodes leave; decisions that still matter move, dated, to `systemPatterns.md`.
6. **Verbose?** Compact prose into facts without losing meaning. Reasoning stays when it
   binds future decisions ("new persistence tech needs architecture approval"), goes when
   it only narrates the past.
7. **Over budget?** `activeContext.md` against its size limit in `AGENTS.md`; maps against
   their 40 lines. Over budget triggers the checks above — never blind truncation.

Estimate context cost as tokens ≈ bytes ÷ 4 and say it is an estimate — one command for the
whole scope (e.g. `git ls-files '.memory-bank/*.md' '.architecture/*.md' | xargs wc -c`),
no other tooling.

**Never remove knowledge just to save tokens.** Always preserved: architectural decisions
and constraints, business rules, security requirements, compatibility notes, non-obvious
dependencies, project conventions, anything not reliably rediscoverable. Unsure whether
something is safe to drop → keep it and list it for the user's review.

## Phase 2 — Plan and gate

Present the cleanup plan before touching anything:

```text
FeatherSpec Context Cleanup
Files analyzed: 7 · already concise: 4
~ compact: activeContext.md (finished episodes) · techContext.md (verbose prose)
! duplicate: "modular monolith" in systemPatterns.md + techContext.md → canonical: systemPatterns.md
? verify: techContext.md names Redis — not found in the repository
Report-only: 0007-user-login.plan.md handoff looks finished (its own command edits it)
Estimated: 5,900 → ~3,800 tokens (−36 %) · preserved: decisions, constraints, conventions
Apply? (yes / details <file> / abort)
```

One yes covers the listed edits; every `?` line is applied only after its own answer, and
that answer covers the fact everywhere it appears. Nothing outside the plan gets written.

## Phase 3 — Apply

Replace, merge, consolidate, shorten, move — never append a corrected version below an old
one. Keep each file's structure as its rule defines it; refresh (or add) the `Last updated`
line of every file you touch. Snapshot drift found in Phase 1: run
`/sdd-architecture-update` exactly as if the user had typed it, with your findings as the
observed state — do not edit the snapshot here. A never-initialized snapshot (`TBD` values)
is a `/sdd-architecture-scan` recommendation, not drift.

## Phase 4 — Validate and report

Re-read every changed file: it still answers its purpose in 30 seconds, no preserved-list
item was lost, no meaning inverted. Re-estimate sizes and report: files analyzed/changed,
tokens before → after with the reduction, what was removed (duplicate · stale ·
rediscoverable · historical), what was preserved, report-only findings, and a one-word
context health. Then propose one commit (git writes stay behind the Ask-first gate in
`AGENTS.md`).

Re-running after a cleanup is free and safe: already-optimized files report "already
concise" and nothing is written. Never re-compact content that already states one fact per
line just to reword it — stability is part of the contract.
