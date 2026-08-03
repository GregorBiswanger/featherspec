# Migration Brief — from Setup Brief v1 to the shared-source layout

**Status:** Implemented

> This spec was provided as `MIGRATION-BRIEF.md`, executed in full, and moved here. It
> supersedes the wiring described in [0001-bootstrap-featherspec.md](0001-bootstrap-featherspec.md):
> the separate Copilot `.github/prompts/` + `.github/instructions/` + `copilot-instructions.md`
> and the Claude `.claude/commands/` are gone, replaced by shared `.claude/skills/` and
> `.claude/rules/` that both tools read.

---

## Context

The repository was first built from `SETUP-BRIEF.md` (v1), which assumed both tools need
separate wiring. That assumption was wrong: VS Code Copilot reads most of `.claude/` natively.
This migration removed the duplication v1 created.

## 1. Interop matrix — verified before migrating

What GitHub Copilot in VS Code reads from `.claude/`, verified against current official docs
(VS Code 1.109+ agent-customization pages and the Copilot settings reference; Claude Code
skills and memory docs).

### Shared — one file, both tools

| Artifact | Claude Code | VS Code Copilot | Caveats |
| --- | --- | --- | --- |
| `CLAUDE.md` | native | native, `chat.useClaudeMdFile` | Copilot does not expand Claude's `@file` import (undocumented but assumed; we disable the setting anyway) |
| `.claude/rules/*.md` with `paths:` | native, loads on matching file read | native — reads `paths` instead of `applyTo`; defaults to `**` when omitted | **NOT a default location** for a workspace `.claude/rules`; must be enabled in `.vscode/settings.json` |
| `.claude/skills/<name>/SKILL.md` | native, `/name` | native — default project-skills location | directory name must equal frontmatter `name`, else it silently fails to load in VS Code |
| `.claude/agents/*.md` | native (subagents) | native — detected as Claude sub-agents format | semantics differ (persona vs isolated worker) |
| `hooks` in `.claude/settings.json` | native | native — parses Claude's format | VS Code ignores matcher values; hooks must filter on `tool_name` in the script |

### Not shared — stays separate

| Artifact | Claude Code | VS Code Copilot |
| --- | --- | --- |
| Slash-command prompt files | `.claude/commands/*.md` | `.github/prompts/*.prompt.md` (different folder and extension) |
| `` !`shell` `` injection, `$ARGUMENTS`, `$0`/`$1` | supported | not supported (would appear as literal text) |
| `AGENTS.md` | via `@AGENTS.md` import | native, `chat.useAgentsMdFile` |
| `.claude/settings.json` keys except `hooks` | native | ignored |
| MCP servers | `.mcp.json`, `mcpServers` | `.vscode/mcp.json`, `servers` |
| Auto memory | `~/.claude/projects/<project>/memory/` | no equivalent |
| Output styles, plugins/marketplaces | Claude-specific | Copilot-specific |

### Deviations found during verification (reported, not worked around)

- **`.claude/rules` is not a default Copilot location.** The default of
  `chat.instructionsFilesLocations` is `{ ".github/instructions": true, "~/.claude/rules": false }`.
  Workspace `.claude/rules` is not present and user `~/.claude/rules` is disabled. The shipped
  `.vscode/settings.json` enables `.claude/rules` explicitly. The `paths:`/`**`-default behavior
  is confirmed.
- **`chat.agentFilesLocations` default is `.github/agents` only** (not `.claude/agents`).
  `.claude/agents/*.md` is still detected natively; the setting default just does not list it.
  Irrelevant here — the persona is kept Copilot-only in `.github/agents`.
- **Skills "Agent mode only" is undocumented.** VS Code docs say skills are "available in chat
  and agent mode." The README recommends Agent mode (where workflow tool-actions run) without
  asserting an Ask/Edit exclusion as fact.
- **CLAUDE.md `@import` non-expansion is undocumented.** Does not matter: `.vscode/settings.json`
  sets `chat.useClaudeMdFile: false`, so Copilot ignores `CLAUDE.md` and reads `AGENTS.md`.
- **`chat.hookFilesLocations` default is documented inconsistently** between two official pages
  (`{}` vs a populated object). We ship no hooks; noted as a caveat only.
- **`chat.agentSkillsLocations` default already includes `.claude/skills: true`**, so skills work
  in Copilot out of the box; `.vscode/settings.json` sets it anyway to document the dependency.

No matrix row was contradicted in a way that breaks the design.

## 2–5. Migration performed

- **Commands → shared skills.** The eight workflows collapsed into
  `.claude/skills/sdd-<name>/SKILL.md`, based on the richer Claude command bodies. Removed all
  `` !`shell` `` injection and `$ARGUMENTS`/`$0`/`$1`; rewrote them as instructions. Added
  `name: sdd-<name>` (equal to the directory), `disable-model-invocation: true`, and a top-of-body
  comment noting the deliberate absence of injection/argument syntax. Deleted `.claude/commands/`
  and `.github/prompts/`.
- **Instructions → rules only.** Deleted `.github/instructions/`. `.claude/rules/*.md` are read by
  both tools; each `paths:` is a YAML list and matches real paths.
- **Constitution.** Deleted `.github/copilot-instructions.md` (Copilot reads `AGENTS.md` natively).
  `CLAUDE.md` reduced to the single line `@AGENTS.md`. `AGENTS.md` still holds `DocLanguage`, the
  `architecture:` snapshot, and *Style & Output Preferences*, with no copy elsewhere.
- **`.vscode/settings.json`** added to make the interop explicit.
- **`/sdd-eject` deleted.** With this layout there is nothing to eject; both tools read the same
  files. The README documents both one-tool conversion tables instead.

## 4. What stays tool-specific

- `.github/agents/SpecDrivenAgent.agent.md` — Copilot-only persona (persona vs isolated-subagent
  semantics). Claude Code gets the same persona via `AGENTS.md` + `/sdd-overview`.
- MCP, hooks, and auto memory — none shipped; documented in the README for later.

## 6. Definition of Done

- [x] Every row of the section 1 matrix verified against current documentation; deviations
      reported above, not worked around.
- [x] Eight directories under `.claude/skills/`; each `SKILL.md` has `name` = directory name, a
      `description`, and `disable-model-invocation: true`; every frontmatter block parses as YAML.
- [x] No shell injection or argument variables remain under `.claude/`.
- [x] `.claude/commands/`, `.github/prompts/`, `.github/instructions/` and
      `.github/copilot-instructions.md` no longer exist.
- [x] `.github/` contains only `agents/SpecDrivenAgent.agent.md`.
- [x] `CLAUDE.md` is exactly one line.
- [x] No rule or workflow text exists twice among the live wiring files.
- [x] Every `paths:` value in `.claude/rules/` is a YAML list and matches real paths.
- [x] README covers the matrix, Agent mode, the deliberate constraints, both eject tables, and
      the troubleshooting entries.
- [x] Completion summary (files deleted/created, unconfirmable rows) printed at hand-off.
