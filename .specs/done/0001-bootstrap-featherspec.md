# Setup Brief — FeatherSpec

**Status:** Implemented

> This is the template's own bootstrap spec. It was provided as `SETUP-BRIEF.md`, implemented
> in full, and moved here as the first completed spec.

---

## 1. Goal

Build **FeatherSpec**: a featherweight Spec-Driven Development (SDD) template with a
Memory Bank, usable with **Claude Code** and **GitHub Copilot** — hybrid by default,
reducible to a single tool with one command.

Non-goals: no CLI, no Node/Python runtime, no build step, no dependencies. Markdown and
folders only. If a feature cannot be expressed in Markdown plus native tool
configuration, it does not belong in this template.

## 2. Source material

The predecessor is public and is the content source for everything except the wiring:

```bash
git clone --depth 1 https://github.com/GregorBiswanger/copilot-spec-driven-template /tmp/fs-source
```

Reuse the substance of these files verbatim where possible — the wording is deliberate
and battle-tested. Only adapt paths, command names, and tool-specific syntax:

| Source | Reuse for |
| --- | --- |
| `.github/copilot-instructions.md` | the shared constitution (`AGENTS.md`) |
| `.github/instructions/*.instructions.md` | Copilot instructions **and** Claude rules |
| `.github/prompts/*.prompt.md` | Copilot prompts **and** Claude commands |
| `.github/agents/SpecDrivenAgent.agent.md` | Copilot agent **and** the `/sdd-overview` command |
| `.github/memory-bank/*`, `.github/specs/*` | copied unchanged |

## 3. Architecture: one brain, two hoods

The single most important rule: **no rule is written down twice.**

```
AGENTS.md                        ◀ THE constitution. Tool-neutral. Single source of truth.
├── CLAUDE.md                    → thin loader: `@AGENTS.md` + Claude-only wiring
└── .github/copilot-instructions.md
                                 → thin loader: points at AGENTS.md + Copilot-only wiring
.specs/                          ◀ shared data: backlog/ active/ done/
.memory-bank/                    ◀ shared data: projectbrief, systemPatterns,
                                              activeContext, techContext
.claude/rules/*.md               Claude wiring   (paths: frontmatter)
.claude/commands/sdd-*.md        Claude wiring   (/sdd-* commands)
.github/instructions/*.md        Copilot wiring  (applyTo: frontmatter)
.github/prompts/sdd-*.prompt.md  Copilot wiring  (/sdd-* prompts)
.github/agents/*.agent.md        Copilot wiring  (SpecDrivenAgent chat mode)
```

### Why the data moved out of `.github/`

In the predecessor, specs and the Memory Bank lived under `.github/`. That was correct
for a Copilot-only template and wrong for a hybrid one: a user who ejects the Copilot
side would delete `.github/` and lose their specs. Move them to top-level `.specs/` and
`.memory-bank/`. Both are plain Markdown, so neither tool cares about the location — but
ejection becomes a safe, single command.

Add `.specs/README.md` explaining the lifecycle folders, and keep `.gitkeep` files in
the three empty spec folders.

### Why `AGENTS.md` and not `copilot-instructions.md`

`AGENTS.md` is the emerging cross-tool standard and, more importantly, it is
tool-neutral: it survives either ejection path unchanged.

- **Claude Code** reads it through `@AGENTS.md` on the first line of `CLAUDE.md`.
  The import is expanded into context at session start. This is a documented, reliable
  mechanism.
- **GitHub Copilot** reads `.github/copilot-instructions.md` natively. Recent VS Code
  versions also read `AGENTS.md` directly; verify this and note the relevant setting in
  the README. Regardless of that setting, `.github/copilot-instructions.md` must open
  with an explicit instruction to read `AGENTS.md` before starting any task, and must
  restate the three or four hardest non-negotiables inline so the setup degrades
  gracefully if the file is never opened.

**Everything mutable lives in `AGENTS.md` only:** `DocLanguage`, the `architecture:`
snapshot, and the *Style & Output Preferences* section. Every command that updates those
writes to `AGENTS.md`. Neither loader file may contain a copy — that is the drift the
whole design exists to prevent. State this rule inside both loader files so the agents
themselves enforce it.

## 4. Command parity

Both tools expose **identical command names**, so the workflow reads the same in
documentation, videos, and talks regardless of which tool a user runs:

| Command | Purpose |
| --- | --- |
| `/sdd-overview` | Workflow overview, current spec status, command list |
| `/sdd-setup` | Onboarding wizard: sets `DocLanguage`, seeds Memory Bank, first architecture snapshot |
| `/sdd-specify` | Idea → spec skeleton with testable acceptance criteria |
| `/sdd-plan` | Spec → step-by-step implementation plan |
| `/sdd-compile` | Readiness check: tests, acceptance criteria, docs sync |
| `/sdd-architecture-update` | Detect drift, update snapshot + Memory Bank (with confirmation gate) |
| `/sdd-lifecycle` | Spec status and moves between backlog/active/done |
| `/sdd-style-update` | Capture coding style preferences into `AGENTS.md` |
| `/sdd-eject` | Reduce the repository to a single tool (see section 6) |

The `sdd-` prefix is mandatory on both sides: it prevents shadowing Claude Code's
built-in commands and bundled skills, and it groups everything into one autocomplete
namespace.

### Claude Code specifics

- Location: `.claude/commands/sdd-<name>.md`. The **filename** is the command name.
- Frontmatter: `description` (required, shown in the `/` menu) and `argument-hint`.
- Use `$ARGUMENTS`, or `$0`/`$1` for positional arguments, instead of the
  `param=<value>` usage blocks the predecessor used in prose.
- Use `` !`command` `` (inline) shell injection to put facts into the prompt **before**
  the model sees it, rather than instructing the model to go look. Applied to at least:
  - `/sdd-overview`: `DocLanguage` line from `AGENTS.md`, contents of all three spec
    folders, `git status --short`
  - `/sdd-lifecycle`: contents of all three spec folders
  - `/sdd-architecture-update`: `find` of the directory tree at depth 2, plus the current
    `architecture:` block extracted from `AGENTS.md`
  - `/sdd-compile`: `git status --short` and recent `git log --oneline`
  - `/sdd-specify`: existing spec filenames, to avoid duplicates
  Every injected command degrades gracefully outside a git repository or with empty
  folders (`2>/dev/null || echo '(none)'`) and was executed in a shell before committing.
- Do **not** use skills. Custom commands cover this use case with zero context cost and
  no automatic invocation. Reasoning noted in the README.
- Do **not** map `SpecDrivenAgent` onto `.claude/agents/`. Subagents are isolated
  workers with their own context window, not personas. The persona lives in
  `AGENTS.md` (always loaded) plus `/sdd-overview` (the greeting and command list).

### Claude Code path-scoped rules

`.claude/rules/*.md` with a `paths:` frontmatter list is the exact counterpart of
Copilot's `applyTo:`. One rule per topic, mirroring the Copilot instructions one-to-one:

| Rule file | `paths:` | Copilot counterpart |
| --- | --- | --- |
| `constitution.md` | `AGENTS.md` | `.github/instructions/constitution.instructions.md` |
| `memory-bank.md` | `.memory-bank/**/*.md` | `memory-bank.instructions.md` |
| `specs.md` | `.specs/**/*.md` | `specs.instructions.md` |
| `repo-docs.md` | `README.md`, `docs/**/*.md` | `repo-docs.instructions.md` |

Known limitation designed around: path-scoped rules load when a **matching file is
read**, not on every tool call. When `/sdd-specify` creates a brand-new spec, the spec
rule may not be loaded yet. Therefore the essentials — `DocLanguage`, the `**Status:**`
line, the section skeleton — are also stated inside `/sdd-specify` itself, and this is
documented in the README so future contributors do not "clean up" the duplication.

### Settings

`.claude/settings.json` with `"autoMemoryEnabled": false`. Claude Code's auto memory is
a machine-local, per-repository note store — effectively a second, invisible Memory Bank
next to the version-controlled one. This template keeps exactly one visible source of
truth. Explained in the README, with instructions to re-enable it.

## 5. Content rules

- All template files are written in **English**. `DocLanguage` governs the *user's*
  project documentation (Memory Bank, specs, README), not this template's own wiring.
- Keep `CLAUDE.md` under 40 lines. It is wiring, not content.
- Keep `AGENTS.md` under 200 lines. If it grows, move material into a path-scoped rule.
- No secrets, no `.env` examples, no tokens anywhere.

## 6. `/sdd-eject` — choosing one world

Hybrid is the default, but a user must be able to commit to a single tool without
manual archaeology. Shipped on both sides. It asks which tool to keep, then:

**Keep Claude Code only**
- delete `.github/copilot-instructions.md`, `.github/instructions/`, `.github/prompts/`,
  `.github/agents/`
- keep `AGENTS.md` as-is (`CLAUDE.md` still imports it)
- rewrite the README: remove the Copilot sections and the parity table

**Keep GitHub Copilot only**
- delete `.claude/` and `CLAUDE.md`
- keep `AGENTS.md` as-is (`.github/copilot-instructions.md` still points at it)
- rewrite the README: remove the Claude Code sections and the parity table

In both cases `AGENTS.md`, `.specs/` and `.memory-bank/` are untouched — that is the
whole point of the neutral layout. The command shows the file list and asks for
confirmation before deleting anything, and it never touches `.git/`.

## 7. README requirements

Written for someone who has never seen the predecessor. Covers: what it is and is not;
repository layout with the shared/tool-specific split; side-by-side quick start; the
command parity table; how the two stay in sync; the `/sdd-eject` path with manual `rm`
equivalents; adding your own commands and rules for both tools; agent selection and the
deliberate non-choices (skills, subagents, output styles); the Memory Bank pattern; a
troubleshooting table; a "coming from `copilot-spec-driven-template`?" section; and the
MIT license reference. Tone: direct and technical.

## 8. Definition of Done

- [x] Every file in section 3 exists; nothing from the predecessor was silently dropped.
- [x] `AGENTS.md` contains `DocLanguage`, the `architecture:` snapshot, and the Style &
      Output Preferences section. Neither `CLAUDE.md` nor
      `.github/copilot-instructions.md` contains a copy of any of them.
- [x] Every `.claude/**/*.md` file has valid YAML frontmatter — parsed to confirm.
- [x] Every shell injection was executed in a shell; output verified; degrades
      gracefully in an empty repo and outside git.
- [x] The nine commands exist with identical names on both sides.
- [x] Both `/sdd-eject` paths were dry-run reviewed: after either, no dangling reference
      to the removed tool remains in `AGENTS.md`, `.specs/`, or `.memory-bank/`.
- [x] `grep -ri "copilot-spec-driven-template"` returns only the intentional migration
      note in the README (and this bootstrap spec).
- [x] README covers all twelve points from section 7.
- [x] Repository description and topics suggested for GitHub:
      `spec-driven-development`, `claude-code`, `github-copilot`, `memory-bank`,
      `template`, `sdd`, `ai-agents`, `generative-ai`.
- [x] This brief has been moved to `.specs/done/0001-bootstrap-featherspec.md`, with a
      `**Status:** Implemented` line.

## 9. Open decisions (resolved)

- Project name: **FeatherSpec** (default kept).
- Command prefix `sdd-` (default kept).
- CI drift check: **not shipped** — it contradicts the featherweight promise. Mentioned
  in the README as an optional idea instead.
