# Changelog

All notable changes to the FeatherSpec template are documented here.

> In a derived project this file is a snapshot at adoption — the template never updates it
> there. The living changelog is the template's
> [GitHub Releases page](https://github.com/GregorBiswanger/featherspec/releases).

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) · Versioning:
template-semver — **MAJOR** means derived projects need a real migration step, **MINOR**
means additive capability that merges into a customized project, **PATCH** means wording and
docs fixes that are safe to overwrite.

## [1.4.0] - 2026-08-27

### Added

- `/sdd-clean`: context cleanup for FeatherSpec's persistent markdown — analyzes every
  managed file against its declared purpose, detects verbosity, duplicates, stale and
  code-rediscoverable content, previews a cleanup plan behind one gate, compacts
  semantically (decisions, constraints and conventions are always preserved), and reports
  estimated tokens before → after. Snapshot drift hands over to
  `/sdd-architecture-update`'s gate; specs and plans get report-only findings. Re-running
  is safe: already-optimized files are left untouched.
- `/sdd-architecture-update` recommends `/sdd-clean` when the Memory Bank has visibly
  outgrown its purpose — cleanup itself never runs as a side effect.

## [1.3.0] - 2026-08-27

### Added

- `/sdd-plan` opens with a software-architect persona and, where the tool supports
  subagents, delegates each research lookup to an isolated agent — only distilled,
  source-backed findings enter the planning context.
- Plans name their **quality gates**: the Approach block lists the test/build/lint commands
  from `techContext.md`, step Verify lines draw on them, and the final step runs them all.
- Bidirectional spec ↔ plan linkage: specs carry a `**Plan:**` line under their status,
  maintained by `/sdd-plan`; plans already linked their spec.
- `/sdd-lifecycle` gains the reactivation move: an `Implemented` spec whose behaviour
  changes moves back to `active/` after `/sdd-plan` Mode C's impact report.

### Changed

- The plan's Approach block records chosen technologies with a one-line rationale each;
  long-lived decisions go dated to `systemPatterns.md`.

## [1.2.0] - 2026-08-26

### Added

- Versioning: the template carries a `FeatherSpecVersion:` stamp in `AGENTS.md`'s managed
  settings block; releases are annotated git tags surfaced as GitHub Releases, recorded here
  and in the version ledger.
- `/sdd-featherspec-update`: check the installed template version (`check` mode works even
  on customized, unstamped projects via feature probes) and update safely — fetches the
  recorded base and the target release as real trees, classifies every file by canonical-hash
  three-way comparison, previews everything before writing, migrates around preserved
  customizations, surfaces conflicts instead of choosing sides, keeps a rollback point,
  validates the result mechanically, and moves the version stamp only after success.
  Resumable at any point; re-running after success is a free health check.
- Version ledger: an append-only appendix in the update command's body recording
  per-release semantic migration knowledge back to 0.1.0 — renames, key migrations, slot
  edits, data notes, semantic flips, and version probes for unstamped projects.

### Changed

- `AGENTS.md`: the managed settings block gains the version stamp and is now managed by
  `/sdd-setup` and `/sdd-featherspec-update`; the `DocLanguage` comment is condensed to one
  line to keep the constitution under its line target; `.sdd-update/` joins the volatile,
  gitignored working locations.

## [1.1.0] - 2026-08-26

### Added

- `/sdd-architecture-scan`: deep, resumable, budget-cascaded analysis of an existing
  codebase producing an architecture fingerprint — brownfield onboarding for both Claude
  Code and GitHub Copilot.
- `sdd-scout` explorer subagent for both tools (`.claude/agents/sdd-scout.md`,
  `.github/agents/sdd-scout.agent.md`).
- Path-scoped rule for curated module maps (`.claude/rules/architecture-map.md`).
- Scan-only locations: `.architecture/` (curated module maps) and `.sdd-scan/` (volatile
  working state, gitignored).
- `/sdd-setup` asks new-vs-existing project and routes existing software into the deep scan.
- The architecture snapshot header tracks the last deep scan alongside the last
  reconciliation.

### Changed

- `/sdd-architecture-update` recommends `/sdd-architecture-scan` instead of guessing when
  drift is too large or the area is unmapped.
- Constitution eviction rules: gates, ask-first boundaries and non-negotiables may never be
  evicted from always-loaded context.
- README: brownfield onboarding path documented up to enterprise scale.

## [1.0.0] - 2026-08-04

### Added

- Always-loaded **Progress & state sync gate**: plan checkbox, traceability row and
  `activeContext.md` must move in the same change set as the code; "done" requires plan,
  active context and code to agree.
- `projectbrief.md` wired into the working loop: `/sdd-specify` and `/sdd-compile` read and
  reconcile against it.

### Changed

- Architecture updates now run **unprompted**: agents execute `/sdd-architecture-update`
  themselves in the same change set on structural drift (previously: propose and wait).
- All commands hardened against audit findings: tightened gates in `/sdd-clarify`,
  `/sdd-compile`, `/sdd-lifecycle`, `/sdd-plan`, `/sdd-specify`, `/sdd-setup`,
  `/sdd-style-update`.
- Preference capture triggers in any language and on "remember this"-style requests.

### Removed

- `docs/history/` (the template's own build specs): the template ships fully blank.

## [0.4.0] - 2026-08-03

### Added

- `/sdd-clarify`: adversarial pass over a finished spec — contradictions, ambiguity,
  untestable criteria, implementation posing as intent, missing failure modes — ending in
  one question.
- Closed verification chain: criterion → step → code → test → `Verified:` → verdict. Plan
  steps carry a `Verified:` field; traceability gains a Test column and the state vocabulary
  `open | built | verified`.
- Five acceptance-criterion shapes (always-true, event, state, unwanted behaviour, optional
  feature) with shall/should discipline and a falsifiability test.
- Non-negotiables gain an **Ask first** tier and a documented **fast path** for changes
  smaller than their spec.
- Project logo.

### Changed

- `/sdd-compile` re-derives readiness from the repository, not from ticked checkboxes, and
  returns NOT READY when the suite did not run.
- `/sdd-lifecycle` requires recorded evidence before a spec moves to `done/`.
- Copilot persona reduced to a thin loader; drifted rule copies removed.
- README rewritten for fast onboarding; detail moved to the wiki.
- The template's own build specs moved out of `.specs/done/` so a fresh clone starts with an
  empty spec numbering.

## [0.3.0] - 2026-08-03

### Added

- Persisted plans: `/sdd-plan` writes `NNNN-slug.plan.md` beside its spec — baby steps with
  per-step Verify lines, a current-step pointer, a handoff block and a
  criteria→steps→code traceability table; three modes (plan, resume, re-plan).
- Plan upkeep rule (`.claude/rules/plans.md`); `/sdd-lifecycle` moves spec and plan as a
  pair.
- `.github/prompts/*.prompt.md` thin loaders giving GitHub Copilot slash commands.

### Changed

- **Breaking (pre-1.0):** command bodies moved from `.claude/skills/*/SKILL.md` to
  `.claude/commands/*.md`; skills removed entirely so no command description is ever
  advertised to the model. `.vscode/settings.json` swaps `chat.agentSkillsLocations` for
  `chat.promptFilesLocations`.

## [0.2.0] - 2026-08-03

### Changed

- `/sdd-specify` became an adaptive product-owner interview producing a lean spec with
  testable acceptance criteria.

## [0.1.0] - 2026-08-03

### Added

- Initial shared-source hybrid SDD template: one constitution (`AGENTS.md`) loaded by Claude
  Code and GitHub Copilot through thin loaders.
- Eight `/sdd-*` workflows (overview, setup, specify, plan, compile, architecture-update,
  lifecycle, style-update) as shared skills.
- Path-scoped rules (constitution, memory-bank, repo-docs, specs), Memory Bank seed files,
  spec lifecycle folders (`backlog/`, `active/`, `done/`), Copilot agent persona, MIT
  license.

### Fixed

- Interop matrix corrected against current VS Code documentation.
- Rule duplication removed so the single-source promise holds.
- `.gitignore` for local agent configuration.

[1.4.0]: https://github.com/GregorBiswanger/featherspec/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/GregorBiswanger/featherspec/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/GregorBiswanger/featherspec/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/GregorBiswanger/featherspec/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/GregorBiswanger/featherspec/compare/v0.4.0...v1.0.0
[0.4.0]: https://github.com/GregorBiswanger/featherspec/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/GregorBiswanger/featherspec/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/GregorBiswanger/featherspec/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/GregorBiswanger/featherspec/releases/tag/v0.1.0
