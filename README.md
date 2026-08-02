# FeatherSpec

FeatherSpec is a featherweight **Spec-Driven Development (SDD)** template with a Memory Bank
that works with **Claude Code** and **GitHub Copilot** from the *same files*. It is
deliberately **not** a CLI, not a Node or Python package, and has no build step and no
dependencies: Markdown and folders only.

## What it is (and is not)

- **Is:** a repository scaffold that makes an AI coding assistant work spec-first — write a
  spec, plan against it, implement, validate, and keep a small version-controlled Memory Bank
  in sync. The same `/sdd-*` commands exist in both tools because they are the *same skill
  files*, so the workflow reads identically in docs, videos, and talks.
- **Is not:** a program. There is nothing to install or run. If a feature cannot be expressed
  in Markdown plus native tool configuration, it does not belong here.

## Why one set of files works for both tools

VS Code Copilot has grown to read most of Claude Code's configuration natively. FeatherSpec
leans on that overlap: rules, skills, the constitution, and the Memory Bank are single files
that both tools read. Only genuinely tool-specific things stay separate.

### Interop matrix — what GitHub Copilot in VS Code reads from `.claude/`

| Artifact | Claude Code | VS Code Copilot | Caveats |
| --- | --- | --- | --- |
| `AGENTS.md` (constitution) | via `@AGENTS.md` import in `CLAUDE.md` | native, [`chat.useAgentsMdFile`] (default on) | the single source of truth for both |
| `CLAUDE.md` | native | native, `chat.useClaudeMdFile` (default on) — **off** here on purpose | we point Copilot straight at `AGENTS.md`, so the one-line import file is redundant for it |
| `.claude/rules/*.md` (`paths:`) | native, loads when a matching file is read | native — reads `paths:` instead of `applyTo:`; defaults to `**` when omitted | a **default** workspace location, so it works with no settings change; `.vscode/settings.json` states it explicitly to document the dependency and protect users who disabled it in their profile |
| `.claude/skills/<name>/SKILL.md` | native, `/<name>` | native — `.claude/skills` is a default project-skills location, invoked as `/<name>` | frontmatter `name` **must match the parent directory name**, or the skill is not loaded |
| `.github/agents/*.agent.md` | — (kept Copilot-only, see below) | native — `.github/agents` is a default location (`chat.agentFilesLocations`) | a Copilot custom agent is a *selectable persona* |
| `hooks` in `.claude/settings.json` | native (with matchers) | reads `.claude/settings.json`, `.claude/settings.local.json` and `~/.claude/settings.json`; parses Claude's format | VS Code **ignores matcher values** — hooks run on all tool invocations, so a shared hook must filter on `tool_name` in the script |

Sources: VS Code [custom instructions], [agent skills], [custom agents], [hooks], and the
[settings reference]; Claude Code [skills] and [memory/rules].

### Not shared — stays separate

| Artifact | Claude Code | VS Code Copilot |
| --- | --- | --- |
| Slash-command *prompt* files | `.claude/commands/*.md` | `.github/prompts/*.prompt.md` (different folder *and* extension) — **FeatherSpec uses skills instead, so it ships neither** |
| `` !`shell` `` injection, `$ARGUMENTS`, `$0`/`$1` | supported | **not** supported — the documented mechanism is `${input:…}` — so the shared skills use neither |
| Everything in `.claude/settings.json` except `hooks` (`autoMemoryEnabled`, `permissions`, …) | native | no documented support (treat as ignored) |
| MCP servers | `.mcp.json`, root key `mcpServers` | `.vscode/mcp.json`, root key `servers` |
| Auto memory | `~/.claude/projects/<project>/memory/` | no equivalent |
| Output styles, plugins/marketplaces | Claude-specific | Copilot-specific |

[`chat.useAgentsMdFile`]: https://code.visualstudio.com/docs/copilot/reference/copilot-settings
[custom instructions]: https://code.visualstudio.com/docs/agent-customization/custom-instructions
[agent skills]: https://code.visualstudio.com/docs/agent-customization/agent-skills
[custom agents]: https://code.visualstudio.com/docs/agent-customization/custom-agents
[hooks]: https://code.visualstudio.com/docs/agent-customization/hooks
[settings reference]: https://code.visualstudio.com/docs/copilot/reference/copilot-settings
[skills]: https://code.claude.com/docs/en/skills
[memory/rules]: https://code.claude.com/docs/en/memory

## Repository layout

```
AGENTS.md                          constitution — Copilot native, Claude via CLAUDE.md
CLAUDE.md                          one line: @AGENTS.md

.claude/
  rules/*.md                       SHARED path-scoped rules (paths:)
  skills/sdd-*/SKILL.md            SHARED workflows (/sdd-* in both tools)
  settings.json                    Claude-only keys (autoMemoryEnabled); Copilot ignores them

.github/
  agents/SpecDrivenAgent.agent.md  Copilot-only persona (see "Selecting the agent")

.vscode/settings.json              makes the interop explicit

.specs/                            shared data: backlog/ active/ done/  (+ README)
.memory-bank/                      shared data: projectbrief, systemPatterns,
                                                activeContext, techContext
README.md, LICENSE
```

## Quick start

### Claude Code

1. Open the repository. `CLAUDE.md` imports `AGENTS.md` (`@AGENTS.md`) at session start, so the
   constitution is always loaded.
2. Run `/sdd-overview` for the workflow map, or `/sdd-setup` to onboard a new project.
3. New skill folders under `.claude/skills/` are picked up without a restart.

### GitHub Copilot (VS Code)

1. Use Copilot in **Agent mode**. The docs list skills as "available in chat and agent mode";
   Agent mode is the one where these workflows can actually run tools and edit files.
2. `AGENTS.md`, `.claude/rules`, and `.claude/skills` are all default locations, so this works
   on a fresh clone. The shipped `.vscode/settings.json` restates them explicitly.
3. Optionally select the **SpecDrivenAgent** persona from the agent picker, then run
   `/sdd-overview` or `/sdd-setup`.

## Commands

Both tools expose the same `/sdd-*` commands because they are the same skill files under
`.claude/skills/`:

| Command | Purpose |
| --- | --- |
| `/sdd-overview` | Workflow overview, current spec status, command list |
| `/sdd-setup` | Onboarding wizard: sets `DocLanguage`, seeds Memory Bank, first architecture snapshot |
| `/sdd-specify` | Idea → spec skeleton with testable acceptance criteria |
| `/sdd-plan` | Spec → step-by-step implementation plan |
| `/sdd-compile` | Readiness check: tests, acceptance criteria, docs sync |
| `/sdd-architecture-update` | Detect drift, update snapshot + Memory Bank (confirmation gate) |
| `/sdd-lifecycle` | Spec status and moves between backlog/active/done |
| `/sdd-style-update` | Capture coding style preferences into `AGENTS.md` |

The `sdd-` prefix keeps the commands from shadowing Claude Code's built-in commands and groups
them into one autocomplete namespace. Each skill has `disable-model-invocation: true` — these
are workflows with side effects, so they run only when you invoke them, never auto-triggered.

## How the two stay in sync

The rule: **no rule and no workflow is written down twice** — apart from two reinforcements
listed below, which are deliberate and documented.

- **`AGENTS.md` is the only place mutable state lives** — `DocLanguage`, the `architecture:`
  snapshot, and *Style & Output Preferences*. Every command that changes those writes to
  `AGENTS.md`. `CLAUDE.md` is one line and holds no copy.
- **Rules and skills are single files both tools read.** No `.github/instructions/`, no
  `.github/prompts/`, no `.github/copilot-instructions.md` — they would only duplicate what
  `.claude/rules/` and `.claude/skills/` already provide.
- **`AGENTS.md` states policy; rule files state craft.** `AGENTS.md` defines the spec
  lifecycle and the Memory Bank file set. `.claude/rules/specs.md` and
  `.claude/rules/memory-bank.md` cover only how to *write* those files, and link to
  `AGENTS.md` for the policy. `.specs/README.md` is orientation and links rather than
  restating.

### The two documented exceptions

Both exist because a path-scoped rule loads only when a **matching file is read**, which can
be too late for a skill that is creating or moving that very file:

| Reinforcement | Where | Why |
| --- | --- | --- |
| Spec essentials (`DocLanguage`, the `**Status:**` line, the section skeleton) | `sdd-specify/SKILL.md` | a brand-new spec has not been read, so `.claude/rules/specs.md` may not have loaded yet |
| Spec lifecycle safety rules (one folder at a time, delete the original, duplicate check) | `sdd-lifecycle/SKILL.md` | this skill performs the moves and deletions; the safety rules must be in front of the model as it acts |

In both cases `AGENTS.md` remains authoritative: if a copy ever diverges, fix the copy. Do not
"clean up" these two by deleting them.

### Deliberate constraints in the shared skills

Because one skill file is read by both tools, the skills are written to the lowest common
denominator. Each constraint has a reason:

- **No `` !`shell` `` injection.** Copilot does not support it; it would land in the prompt as
  literal text. Skills instruct the model to *run* a command and use the result instead.
- **No `$ARGUMENTS` / `$0` / `$1`.** Same reason. Skills say "the user may name a spec path
  after the command; if none is given, list the candidates and ask." Both tools append the
  user's trailing text to the invocation, so the model still sees it.
- **`name` equals the directory name.** Required by VS Code (a mismatch means the skill is not
  loaded) and harmless in Claude Code.

A comment at the top of each `SKILL.md` restates this so nobody reintroduces the syntax later.

## Adding your own skills and rules

Both live under `.claude/` and are read by both tools.

- **Skill:** create `.claude/skills/sdd-mytask/SKILL.md`. The **directory name must equal the
  `name` frontmatter**. Invoked as `/sdd-mytask`. Claude Code picks up new skill folders
  without a restart.

  ```markdown
  ---
  name: sdd-mytask
  description: One-line summary shown in the / menu.
  argument-hint: "[thing]"
  disable-model-invocation: true
  ---
  # /sdd-mytask
  Do something. The user may name a thing after the command; if none, ask.
  ```

  Keep skills free of shell injection and argument variables so they work in Copilot too.

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
  Skills, not subagents, carry the workflows; output styles and plugins are left as
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
| A skill does not appear in VS Code | Confirm the **`name` frontmatter equals the parent directory name** (a mismatch means it is not loaded), and that `name` uses only lowercase letters, numbers, and hyphens — invalid characters make it fail to load silently. |

## Committing to one tool later

Nothing needs to be *ejected* — both tools read the same files. But if you want a repository
with no trace of the other tool, the conversion is mechanical.

### Keep GitHub Copilot only

| From | To | Transformation |
| --- | --- | --- |
| `.claude/skills/<n>/SKILL.md` | `.github/skills/<n>/SKILL.md` | move only; the format is identical |
| `.claude/rules/<n>.md` | `.github/instructions/<n>.instructions.md` | rename, and change frontmatter `paths: [a, b]` to `applyTo: 'a,b'` |
| `AGENTS.md` | `.github/copilot-instructions.md` | move, or keep `AGENTS.md` — both are native |
| `CLAUDE.md` | — | delete |
| `.claude/settings.json` hooks block | `.github/hooks/hooks.json` | restructure to the Copilot hooks file; other keys are dropped |
| `.mcp.json` | `.vscode/mcp.json` | rename root key `mcpServers` to `servers` |
| `.vscode/settings.json` | — | the `.claude/*` location entries become unnecessary |

### Keep Claude Code only

| From | To |
| --- | --- |
| `.github/agents/SpecDrivenAgent.agent.md` | delete, or port to `.claude/agents/` accepting the changed semantics (isolated subagent, not a persona) |
| `.vscode/settings.json` | delete |
| everything else | unchanged |

## Coming from `copilot-spec-driven-template`?

FeatherSpec is the successor to that Copilot-only template. What changed:

- **Data moved out of `.github/`** to top-level `.specs/` and `.memory-bank/`.
- **The constitution moved to `AGENTS.md`** (tool-neutral), read natively by both tools.
- **Commands became shared skills** under `.claude/skills/` with the `sdd-` prefix and
  identical names in both tools (`/setupSpecs` → `/sdd-setup`, `/specify` → `/sdd-specify`, …).
  The separate `.github/prompts/` and `.github/instructions/` folders are gone — one file per
  rule and per workflow now serves both tools.

## License

MIT. See [`LICENSE`](LICENSE).
