# FeatherSpec

FeatherSpec is a featherweight **Spec-Driven Development (SDD)** template with a Memory Bank
that works with **Claude Code** and **GitHub Copilot** from the *same files*. It is
deliberately **not** a CLI, not a Node or Python package, and has no build step and no
dependencies: Markdown and folders only.

## What it is (and is not)

- **Is:** a repository scaffold that makes an AI coding assistant work spec-first — write a
  spec, plan against it, implement, validate, and keep a small version-controlled Memory Bank
  in sync. The same `/sdd-*` commands exist in both tools because each workflow is written
  down *once*, so it reads identically in docs, videos, and talks.
- **Is not:** a program. There is nothing to install or run. If a feature cannot be expressed
  in Markdown plus native tool configuration, it does not belong here.

## Why one set of files works for both tools

VS Code Copilot has grown to read most of Claude Code's configuration natively. FeatherSpec
leans on that overlap: rules, the constitution, the workflow bodies, and the Memory Bank are
single files that both tools read. Only genuinely tool-specific things stay separate.

**No skills on purpose.** A skill advertises itself: both tools read every skill's `name` and
`description` up front so the model can decide to load it. Claude Code drops that advertising
when a skill sets `disable-model-invocation: true`, but VS Code does not document the same
guarantee — so eight workflow descriptions could sit in the system prompt of every single
request. Workflows here are *commands*, which no model is offered: `.claude/commands/<name>.md`
holds the body and Claude Code runs it as `/<name>`; `.github/prompts/<name>.prompt.md` is a
one-line loader that points Copilot at that same body. Reach for a skill only when a workflow
genuinely needs the model to trigger it on its own.

### Interop matrix — what GitHub Copilot in VS Code reads from `.claude/`

| Artifact | Claude Code | VS Code Copilot | Caveats |
| --- | --- | --- | --- |
| `AGENTS.md` (constitution) | via `@AGENTS.md` import in `CLAUDE.md` | native, [`chat.useAgentsMdFile`] (default on) | the single source of truth for both |
| `CLAUDE.md` | native | native, `chat.useClaudeMdFile` (default on) — **off** here on purpose | we point Copilot straight at `AGENTS.md`, so the one-line import file is redundant for it |
| `.claude/rules/*.md` (`paths:`) | native, loads when a matching file is read | native — reads `paths:` instead of `applyTo:`; defaults to `**` when omitted | the two official pages disagree: [custom instructions] lists `.claude/rules` as a default workspace location, while the [settings reference] default value omits it (`{ ".github/instructions": true, "~/.claude/rules": false }`). `.vscode/settings.json` enables it explicitly, so the repo works either way |
| `.claude/commands/<name>.md` (workflow body) | native, `/<name>` from the file name; `disable-model-invocation: true` keeps the description out of context | not read directly — reached through the `.github/prompts` loader below | the body is written once and both tools execute the same text |
| `.github/prompts/<name>.prompt.md` (loader) | — (ignored) | native, `/<name>`; `.github/prompts` is the default location and the `.prompt.md` extension is required | a prompt file is user-invoked only, never offered to the model; each loader is a link to the body and holds no rules |
| Web research from inside a workflow | native `WebSearch` (US-only) and `WebFetch` | built-in `#fetch` for a page, plus the `web` tool in agent mode | `/sdd-plan` asks for "your web search or fetch capability" rather than naming a tool, and requires the model to say so when none is available instead of guessing |
| `.github/agents/*.agent.md` | — (kept Copilot-only, see below) | native — `.github/agents` is a default location (`chat.agentFilesLocations`) | a Copilot custom agent is a *selectable persona* |
| `hooks` in `.claude/settings.json` | native (with matchers) | reads `.claude/settings.json`, `.claude/settings.local.json` and `~/.claude/settings.json`; parses Claude's format | VS Code **ignores matcher values** — hooks run on all tool invocations, so a shared hook must filter on `tool_name` in the script |

Sources: VS Code [custom instructions], [agent skills], [custom agents], [hooks], [agent tools],
and the [settings reference]; Claude Code [skills] and [memory/rules].

### Not shared — stays separate

| Artifact | Claude Code | VS Code Copilot |
| --- | --- | --- |
| The slash-command *entry point* | `.claude/commands/*.md` | `.github/prompts/*.prompt.md` — different folder *and* extension, so one file cannot serve both; FeatherSpec ships a one-line Copilot loader per command and keeps the body in `.claude/commands/` |
| Agent skills (`.claude/skills/<name>/SKILL.md`) | native, `/<name>` | native, `.claude/skills` is a default location | both support it; **FeatherSpec ships none on purpose** — see the note above |
| `` !`shell` `` injection, `$ARGUMENTS`, `$0`/`$1` | supported | **not** supported — the documented mechanism is `${input:…}` — so the shared bodies use neither |
| Everything in `.claude/settings.json` except `hooks` (`autoMemoryEnabled`, `permissions`, …) | native | no documented support (treat as ignored) |
| MCP servers | `.mcp.json`, root key `mcpServers` | `.vscode/mcp.json`, root key `servers` |
| Auto memory | `~/.claude/projects/<project>/memory/` | no equivalent |
| Output styles, plugins/marketplaces | Claude-specific | Copilot-specific |

[`chat.useAgentsMdFile`]: https://code.visualstudio.com/docs/copilot/reference/copilot-settings
[custom instructions]: https://code.visualstudio.com/docs/agent-customization/custom-instructions
[agent skills]: https://code.visualstudio.com/docs/agent-customization/agent-skills
[custom agents]: https://code.visualstudio.com/docs/agent-customization/custom-agents
[hooks]: https://code.visualstudio.com/docs/agent-customization/hooks
[agent tools]: https://code.visualstudio.com/docs/copilot/agents/agent-tools
[settings reference]: https://code.visualstudio.com/docs/copilot/reference/copilot-settings
[skills]: https://code.claude.com/docs/en/skills
[memory/rules]: https://code.claude.com/docs/en/memory

## Repository layout

```
AGENTS.md                          constitution — Copilot native, Claude via CLAUDE.md
CLAUDE.md                          one line: @AGENTS.md

.claude/
  rules/*.md                       SHARED path-scoped rules (paths:)
  commands/sdd-*.md                SHARED workflow bodies — Claude runs them as /sdd-*
  settings.json                    SHARED, version-controlled Claude settings
                                   (settings.local.json is local and git-ignored)

.github/
  prompts/sdd-*.prompt.md          Copilot loaders — one line each, point at the body
  agents/SpecDrivenAgent.agent.md  Copilot-only persona (see "Selecting the agent")

.vscode/settings.json              makes the interop explicit

.specs/                            shared data: backlog/ active/ done/  (+ README)
                                   NNNN-slug.md + NNNN-slug.plan.md per feature
.memory-bank/                      shared data: projectbrief, systemPatterns,
                                                activeContext, techContext
README.md, LICENSE, .gitignore
```

## Quick start

### Claude Code

1. Open the repository. `CLAUDE.md` imports `AGENTS.md` (`@AGENTS.md`) at session start, so the
   constitution is always loaded.
2. Run `/sdd-overview` for the workflow map, or `/sdd-setup` to onboard a new project.
3. New files under `.claude/commands/` are picked up without a restart.

### GitHub Copilot (VS Code)

1. Use Copilot in **Agent mode** — that is where these workflows can run tools and edit files.
2. `AGENTS.md`, `.claude/rules`, and `.github/prompts` are all default locations, so this works
   on a fresh clone. The shipped `.vscode/settings.json` restates them explicitly.
3. Optionally select the **SpecDrivenAgent** persona from the agent picker, then run
   `/sdd-overview` or `/sdd-setup`.

## Commands

Both tools expose the same `/sdd-*` commands because both execute the same body file under
`.claude/commands/`:

| Command | Purpose |
| --- | --- |
| `/sdd-overview` | Workflow overview, current spec status, command list |
| `/sdd-setup` | Onboarding wizard: sets `DocLanguage`, seeds Memory Bank, first architecture snapshot |
| `/sdd-specify` | Adaptive product-owner interview → lean spec with testable acceptance criteria |
| `/sdd-plan` | Spec → persisted baby-step plan file (research, resume, impact analysis) |
| `/sdd-compile` | Readiness check: tests, acceptance criteria, docs sync |
| `/sdd-architecture-update` | Detect drift, update snapshot + Memory Bank (confirmation gate) |
| `/sdd-lifecycle` | Spec status and moves between backlog/active/done |
| `/sdd-style-update` | Capture coding style preferences into `AGENTS.md` |

The `sdd-` prefix keeps the commands from shadowing Claude Code's built-in commands and groups
them into one autocomplete namespace. Each body carries `disable-model-invocation: true` — a
Claude Code command is otherwise model-invocable, and these are workflows with side effects, so
they run only when you invoke them. Copilot prompt files are user-invoked by design, so the
loaders need no equivalent flag.

### From spec to plan to code

`/sdd-specify` writes `.specs/backlog/0007-user-login.md`. `/sdd-plan` writes
`0007-user-login.plan.md` right next to it — same number, same folder, and `/sdd-lifecycle`
moves the pair together. The plan file carries:

- **baby steps** (`T-001`, `T-002`, …), each with what to change and a `Verify:` line that can
  actually be run,
- **the current state** — ticked boxes, a `Current step` pointer and a session handoff block, so
  a new session resumes from the file instead of from your memory,
- **research** with links and retrieval dates, gathered before planning rather than guessed,
- **a traceability table** mapping acceptance criteria → steps → the code paths that were
  actually written.

That table is what makes change cheap. When a requirement moves, `/sdd-plan` reads it in reverse
and reports which steps and which files the change reaches — before anything is edited.

## How the two stay in sync

The rule: **no rule and no workflow is written down twice** — apart from two reinforcements
listed below, which are deliberate and documented.

- **`AGENTS.md` is the only place mutable state lives** — `DocLanguage`, the `architecture:`
  snapshot, and *Style & Output Preferences*. Every command that changes those writes to
  `AGENTS.md`. `CLAUDE.md` is one line and holds no copy.
- **Rules and workflow bodies are single files both tools read.** No `.github/instructions/`,
  no `.github/copilot-instructions.md` — they would only duplicate what `.claude/rules/` and
  `.claude/commands/` already provide. The one Copilot-side folder that does exist,
  `.github/prompts/`, holds loaders: a link and a sentence, never a rule.
- **`AGENTS.md` states policy; rule files state craft.** `AGENTS.md` defines the spec and plan
  lifecycle and the Memory Bank file set. `.claude/rules/specs.md`, `.claude/rules/plans.md`
  and `.claude/rules/memory-bank.md` cover only how to *write* those files, and link to
  `AGENTS.md` for the policy. `.specs/README.md` is orientation and links rather than
  restating.

### The three documented exceptions

All three exist because a path-scoped rule loads only when a **matching file is read**, which
can be too late for a workflow that is creating or moving that very file:

| Reinforcement | Where | Why |
| --- | --- | --- |
| Spec essentials (`DocLanguage`, the `**Status:**` line, the document structure) | `commands/sdd-specify.md` | a brand-new spec has not been read, so `.claude/rules/specs.md` may not have loaded yet |
| Plan essentials (`DocLanguage`, naming, the `**Status:**` line, the file structure) | `commands/sdd-plan.md` | same reason — a brand-new plan file has not been read, so `.claude/rules/plans.md` may not have loaded yet |
| Lifecycle safety rules (one folder at a time, delete the original, duplicate check, plan travels with spec) | `commands/sdd-lifecycle.md` | this workflow performs the moves and deletions; the safety rules must be in front of the model as it acts |

In all three cases `AGENTS.md` remains authoritative: if a copy ever diverges, fix the copy. Do
not "clean up" these by deleting them.

### Deliberate constraints in the shared bodies

Because one body file is executed by both tools, it is written to the lowest common
denominator. Each constraint has a reason:

- **No `` !`shell` `` injection.** Copilot does not support it; it would land in the prompt as
  literal text. Bodies instruct the model to *run* a command and use the result instead.
- **No `$ARGUMENTS` / `$0` / `$1`.** Same reason. Bodies say "the user may name a spec path
  after the command; if none is given, list the candidates and ask." Both tools append the
  user's trailing text to the invocation, so the model still sees it.
- **No tool names in the bodies.** The tool that fetches a web page is `WebFetch` in one and
  `#fetch` in the other, so bodies name the *capability* ("your web search or fetch capability")
  and state what to do when it is missing. Same for terminal work: bodies say which command to
  run and use the result, rather than assuming a tool name.
- **The file name is the command name** in both tools, so `.claude/commands/sdd-plan.md` and
  `.github/prompts/sdd-plan.prompt.md` must share the stem.

A comment at the top of each body restates this so nobody reintroduces the syntax later.

## Adding your own commands and rules

- **Command:** create the body at `.claude/commands/sdd-mytask.md` and the Copilot loader at
  `.github/prompts/sdd-mytask.prompt.md`. Invoked as `/sdd-mytask` in both. Claude Code picks
  up new command files without a restart.

  ```markdown
  ---
  description: One-line summary shown in the / menu.
  argument-hint: "[thing]"
  disable-model-invocation: true
  ---
  # /sdd-mytask
  Do something. The user may name a thing after the command; if none, ask.
  ```

  The loader is the same three fields plus `name: sdd-mytask`, and a single line pointing at
  the body. Keep bodies free of shell injection and argument variables so they work in Copilot
  too.

- **Rule:** create `.claude/rules/mytopic.md` with a `paths:` list (a YAML list even for one
  entry). It loads when a matching file is read.

  ```markdown
  ---
  paths:
    - src/**/*.ts
  ---
  # My topic rules
  - Keep it concise.
  ```

## Selecting the agent

- **GitHub Copilot** has an agent picker — choose **SpecDrivenAgent**
  (`.github/agents/SpecDrivenAgent.agent.md`). This file is kept Copilot-only on purpose. VS
  Code would also read a `.claude/agents/` file, but the semantics differ: in Copilot a custom
  agent is a *persona* you pick from a dropdown; in Claude Code a `.claude/agents/` file is an
  *isolated subagent* with its own context window that does not share your conversation.
- **Claude Code** has no persona picker, and that is deliberate. The persona is `AGENTS.md`
  (always loaded via `@AGENTS.md`) plus `/sdd-overview` (the greeting and command list).
  Commands, not subagents, carry the workflows; output styles and plugins are left as
  project-specific choices, not baked in.

## The Memory Bank pattern

The Memory Bank lives in `.memory-bank/` and is the version-controlled source of truth for
project context. Four files:

| File | What belongs in it |
| --- | --- |
| `projectbrief.md` | mission, primary users, success criteria |
| `systemPatterns.md` | architecture decisions and patterns (long-lived) |
| `techContext.md` | stack, constraints, build/run/test info |
| `activeContext.md` | current focus, active spec, recent changes, decisions in flight, blockers, next steps, validation state |

**Hard rule:** `activeContext.md` stays under **two screen pages**. It is a working/handoff
context, not a changelog or a second spec. If it grows, the content belongs in a more permanent
file — link and summarize instead of duplicating.

### Shared vs. local configuration

`.claude/settings.json` is **shared**: it is version-controlled and applies to everyone who
clones the repository. `.claude/settings.local.json` is **local** — Claude Code writes it when
you grant a permission with "don't ask again", so it holds machine- and user-specific rules,
and VS Code reads hooks from it too. Committing it would push your personal permission set
onto the whole team. The shipped `.gitignore` therefore excludes it, along with
`CLAUDE.local.md` (the documented local-only memory variant, which VS Code also detects).
Put anything the team should share in `.claude/settings.json` and `AGENTS.md`.

### Auto memory is off on purpose

`.claude/settings.json` sets `"autoMemoryEnabled": false`. Claude Code's auto memory is a
machine-local, per-repository note store — effectively a second, invisible Memory Bank. This
template keeps exactly one visible source of truth. To re-enable it, set it to `true` (or
remove the key). Copilot ignores this setting entirely.

## MCP and hooks (ship none, documented for later)

- **MCP:** if you add MCP servers, Claude Code reads `.mcp.json` (root key `mcpServers`) and
  Copilot reads `.vscode/mcp.json` (root key `servers`). The server definitions inside are
  otherwise identical; neither file reads the other, and no setting relocates them.
- **Hooks:** a `hooks` block in `.claude/settings.json` is read by both tools, but VS Code
  **ignores matcher values** (hooks fire on every tool invocation), so any shared hook must
  filter on `tool_name` inside the script. VS Code also uses camelCase hook inputs
  (`tool_input.filePath`) and its own tool names, where Claude Code uses snake_case
  (`tool_input.file_path`) — a shared hook has to handle both. Hooks are a Preview feature in
  VS Code; if yours do not fire, check the `chat.useClaudeHooks` setting.

## Troubleshooting

| Symptom | Try |
| --- | --- |
| Verify what actually loaded (Claude Code) | `/context` |
| Verify what actually loaded (VS Code Copilot) | right-click in the Chat view → **Diagnostics** — lists every loaded agent, skill, and instruction file with load status |
| Manage / inspect Claude Code memory | `/memory` |
| Something in the setup looks broken (Claude Code) | `/doctor` |
| A `.claude/rules` rule did not apply | Confirm a **matching file was read** (rules load on read). In VS Code, confirm nothing set `.claude/rules` to `false` in `chat.instructionsFilesLocations`. |
| A `/sdd-*` command does not appear in VS Code | Confirm the file is `.github/prompts/<name>.prompt.md` — the `.prompt.md` extension is required for discovery — and that nothing removed `.github/prompts` from `chat.promptFilesLocations`. The Chat view's **Agent Debug Log** (ellipsis menu → *Show Agent Debug Logs*) shows prompt-file discovery errors. |
| A `/sdd-*` command does not appear in Claude Code | Confirm the body sits directly in `.claude/commands/` and ends in `.md`; the command name is the file name without the extension. |

## Committing to one tool later

Nothing needs to be *ejected* — both tools read the same files. But if you want a repository
with no trace of the other tool, the conversion is mechanical.

### Keep GitHub Copilot only

| From | To | Transformation |
| --- | --- | --- |
| `.claude/commands/<n>.md` | `.github/prompts/<n>.prompt.md` | move the body into the existing loader file (replacing the pointer line), keep `name`/`description`/`argument-hint`, drop `disable-model-invocation` |
| `.claude/rules/<n>.md` | `.github/instructions/<n>.instructions.md` | rename, and change frontmatter `paths: [a, b]` to `applyTo: 'a,b'` |
| `AGENTS.md` | `.github/copilot-instructions.md` | move, or keep `AGENTS.md` — both are native |
| `CLAUDE.md` | — | delete |
| `.claude/settings.json` hooks block | `.github/hooks/hooks.json` | restructure to the Copilot hooks file; other keys are dropped |
| `.mcp.json` | `.vscode/mcp.json` | rename root key `mcpServers` to `servers` |
| `.vscode/settings.json` | — | the `.claude/*` location entries become unnecessary |

### Keep Claude Code only

| From | To |
| --- | --- |
| `.github/prompts/*.prompt.md` | delete — the loaders exist only for Copilot |
| `.github/agents/SpecDrivenAgent.agent.md` | delete, or port to `.claude/agents/` accepting the changed semantics (isolated subagent, not a persona) |
| `.vscode/settings.json` | delete |
| everything else | unchanged |

## Coming from `copilot-spec-driven-template`?

FeatherSpec is the successor to that Copilot-only template. What changed:

- **Data moved out of `.github/`** to top-level `.specs/` and `.memory-bank/`.
- **The constitution moved to `AGENTS.md`** (tool-neutral), read natively by both tools.
- **Commands got the `sdd-` prefix** and identical names in both tools (`/setupSpecs` →
  `/sdd-setup`, `/specify` → `/sdd-specify`, …). `.github/instructions/` is gone — a single
  `.claude/rules/` file serves both tools. `.github/prompts/` survives as one-line loaders,
  because Copilot has no other way to expose a slash command.

## License

MIT. See [`LICENSE`](LICENSE).
