---
description: Check the installed FeatherSpec version and update safely — fetch a release, migrate, preserve customizations. Resumable.
argument-hint: "[check | target version] (optional — default: latest)"
disable-model-invocation: true
---

<!-- Single source for the /sdd-featherspec-update workflow. Claude Code runs this file
     directly; GitHub Copilot reaches it through the thin loader in
     .github/prompts/sdd-featherspec-update.prompt.md. Deliberately no shell injection and no
     argument-variable substitution: Copilot supports neither. -->

# /sdd-featherspec-update — Template Version Check & Safe Update

Template source: `https://github.com/GregorBiswanger/featherspec` — forks that maintain their
own template change this one line.

The user may type an argument after the command: `check` runs the version check only (no
writes, no fetch unless the user wants the latest-version lookup); a version like `1.3.0`
targets that release; nothing means latest. Speak plainly, in `DocLanguage`, throughout;
wiring you write stays English.

**The safety contract, in one paragraph.** This workflow updates FeatherSpec's own wiring in
this project — never the project's knowledge. It compares three states of every file: **B**ase
(the template version this project came from), **T**arget (the version to update to) and
**P**roject (what is on disk). Base and Target are fetched as real trees, so no maintained
metadata can mis-classify a file. Whatever the comparison cannot prove is asked, never
assumed: *unknown means ask, never overwrite*. User data (`.memory-bank/`, `.specs/`
content, `.architecture/`, `README.md`, `LICENSE`, `CHANGELOG.md`, `docs/`, the logo —
`featherspec-logo.png` or whatever image replaced it — and every `*.local.*` file) is a
**never-write set**: hashed before, proven byte-identical after. The
version stamp moves **last** — an interrupted run always identifies as the old version with
work in flight.

## Where am I? (state router — the files decide, not the conversation)

- Argument `check` → run **Version check** below, report, stop.
- `.sdd-update/_state.md` exists → this is a **resume**. Read `_state.md` first; if it names
  a `governing-body:` path, read that file now and follow **it** for the rest of the run —
  it is the newer updater fetched last session. Resume runs **before** any dirty-tree check.
  If the user asks for a different target than `_state.md` records, say so, discard the state
  and restart. On resume under a different shell than `_state.md` records, re-run the hash
  scripts instead of trusting stored hashes.
- Otherwise → fresh run, **Phase 0**.

Refuse to run when the working tree *is* the template itself (a `featherspec` remote or
folder name **and** no project content beyond the template's own files): the template
updates by git, not by this workflow. When only the remote matches — a project bootstrapped
by cloning the template — say what you see and ask before proceeding.

## Version check (also Phase 0's detection)

1. Locate the constitution: `AGENTS.md`, else `.github/copilot-instructions.md`, else ask.
2. Read `FeatherSpecVersion:` from its managed settings block. Present → that is the
   installed version.
3. Absent → the project predates 1.2.0. Estimate via the **probe ladder** (free, offline,
   from the ledger appendix): `.claude/skills/` exists ⇒ 0.1–0.2 · `.claude/commands/`
   without `sdd-clarify.md` ⇒ 0.3 · `sdd-clarify.md` present but the constitution has no
   *Progress & state sync* section ⇒ 0.4 · sync gate present but no
   `sdd-architecture-scan.md` ⇒ 1.0.0 · scan present ⇒ 1.1.0. Report the estimate as an
   estimate, and that heavy customization can blur it — the precise pass (Phase 1a) can
   confirm when an update actually runs.
4. In `check` mode, offer once: "Look up the newest release? (one network command:
   `git ls-remote --tags https://github.com/GregorBiswanger/featherspec.git`)" — run it only
   on yes, compare, report. Then stop: `check` never writes.

## Phase 0 — Situate (read-only, no network)

Run the version check above to get `V_base` (stamped or estimated-unstamped). Read
`DocLanguage`. Detect the **repo shape** by comparing against the Base tree's per-half file
inventories once Base is fetched (until then, provisionally): *dual* · *Claude-only*
(Copilot half absent) · *Copilot-only* (Claude half absent) · *Copilot-ejected* (bodies
merged into `.prompt.md` files → **advisory mode**, see below). Halves, for every scope
decision: the Claude half is `.claude/`; the Copilot half is `.github/` (prompts,
instructions, agents, `copilot-instructions.md`) plus `.vscode/settings.json`. A half is absent only when Base
ships files for it and the project has none of them; a half hidden by `.gitignore` or
`files.exclude` is present, not deleted. Then check the working tree
(`git status --porcelain`): dirty → ask the user to commit or stash first. Ignore
`.sdd-update/`, this workflow's own freshly installed body and loader (they join the final
commit's path list), and, on resume, every path the worklist already marks applied — any
*other* dirt still blocks. Repositories without git: announce that folder backups replace
the git safety net.

## Phase 1 — Fetch (one gate for all network commands)

Ask once, listing the exact commands (`v<target>` stays symbolic when the target is
*latest* — the newest tag fills it in; the one yes covers the lookup and both clones), then
run on yes:
`git ls-remote --tags https://github.com/GregorBiswanger/featherspec.git` (newest tag = target
unless the user named one), then two shallow clones into the working folder:
`git clone --depth 1 --branch v<target> <url> .sdd-update/target` and the same with
`v<base>` into `.sdd-update/base`. **Before the clones**, create `.sdd-update/` and write
`.sdd-update/.gitignore` containing a single `*` — the folder ignores itself on every
project, tracked or not, and stays out of search results. Local paths work as `<url>` too —
that is how release rehearsals and forks run (git then ignores `--depth`; harmless).

Fallbacks, in order, stated honestly: clone fails → download the two release archives with
your fetch capability and unpack them to `.sdd-update/base` and `.sdd-update/target`; no
fetch capability → create `.sdd-update/` with its `.gitignore` now, write `_state.md`
(base, target, `phase: awaiting-manual-trees`), tell the user which two zips to download
and where to unpack them, then stop — the next invocation resumes from the comparison.
Whenever valid trees already sit in `.sdd-update/base` and `target`, skip fetching and go
straight to validation.

**Validate what arrived** (tags ≥ 1.2.0): each fetched tree's own `FeatherSpecVersion:` must
equal its tag, and the target's ledger appendix must contain an entry for the target version.
Mismatch → abort loudly: "upstream release v<X> is malformed — please report it; not
proceeding on a corrupt baseline."

Write `.sdd-update/_state.md`: base, target, shape, shell/algorithm, phase. From here on,
every write updates `_state.md` and `_worklist.md` in the same change set — a new session
resumes from the files alone.

**Self-upgrade next:** compare the target's copy of this file with the local one, ledger
appendix excluded. If they differ, say so, add `governing-body:
.sdd-update/target/.claude/commands/sdd-featherspec-update.md` to `_state.md` (keep its
other lines), and follow the fetched body from here on. If the *local* body also differs
from **Base**'s copy, the user customized their updater — say that it is set aside for this
run and will appear as a normal conflict in classification. When Base ships no copy at all
(the base predates the updater), a local body is a hand-copied newer version, not a
customization — classification's case 5 handles it.

### Phase 1a — Bootstrap (only when no stamp was found)

The probe estimate picks a candidate base. Confirm it cheaply: fetch the candidate tag (same
gate), materialize the Phase 2 scripts and pass their self-test vector first, then hash
candidate and project and count canonical-hash-identical files **within the shape scope** —
paths under an absent half count neither way. Report transparently ("38 of 41 template
files match v1.0.0 → using it as Base; the 3 divergent files count as your edits — worst
case you get asked, never overwritten"). A poor match (rough guide: under ~80%, or a
near-tie between neighbouring tags) → try the neighbour or show the top candidates and ask.
Offline: the user downloads the candidate's archive; same scoring. Mis-inference toward
*older* is safe by construction — it only ever produces more questions.

## Phase 2 — Classify (deterministic, read-only)

First compose the **rename map** from every ledger entry in `(base, target]`, oldest→newest
(e.g. a 0.2-era `.claude/skills/sdd-plan/SKILL.md` compares against today's
`.claude/commands/sdd-plan.md`). Then hash **once per tree, as one script run** — never file
by file in the terminal. Materialize these two helpers verbatim as `.sdd-update/hash.sh` and
`.sdd-update/hash.ps1`, write one list file naming the union of Base ∪ Target ∪ mapped
project paths ∪ the never-write set as it exists on disk (enumerate `.memory-bank/`,
`.specs/`, `.architecture/`, `docs/`, the root files and every `*.local.*` file — their
before-hashes are what Phase 6's proof re-checks), and run the script for your shell three
times (Base, Target, Project):

```sh
#!/bin/sh
# /sdd-featherspec-update canonical hasher. Usage: sh hash.sh <root> <listfile> <outfile>
root="$1"; list="$2"; out="$3"; tmp="$out.tmp"
bom=$(printf '\357\273\277'); tab=$(printf '\t')
if command -v git >/dev/null 2>&1; then alg=git; else alg=sha256; fi
printf 'algorithm\t%s\n' "$alg" > "$out"
while IFS= read -r p; do
  [ -n "$p" ] || continue
  f="$root/$p"
  if [ ! -f "$f" ]; then printf '%s\tABSENT\n' "$p" >> "$out"; continue; fi
  printf '%s\n' "$(tr -d '\r' < "$f" | sed -e "1s/^$bom//" -e "s/[ $tab]*\$//")" > "$tmp"
  if [ "$alg" = git ]; then h=$(git hash-object "$tmp")
  elif command -v sha256sum >/dev/null 2>&1; then h=$(sha256sum "$tmp" | cut -d' ' -f1)
  else h=$(shasum -a 256 "$tmp" | cut -d' ' -f1); fi
  printf '%s\t%s\n' "$p" "$h" >> "$out"
done < "$list"
rm -f "$tmp"
```

```powershell
# /sdd-featherspec-update canonical hasher. Usage: powershell -File hash.ps1 <root> <listfile> <outfile>
param([string]$Root,[string]$List,[string]$Out)
$tmp = "$Out.tmp"
$git = [bool](Get-Command git -ErrorAction SilentlyContinue)
$lines = @("algorithm`t" + $(if ($git) { 'git' } else { 'sha256' }))
foreach ($p in Get-Content -LiteralPath $List) {
  if (-not $p) { continue }
  $f = Join-Path $Root $p
  if (-not (Test-Path -LiteralPath $f -PathType Leaf)) { $lines += "$p`tABSENT"; continue }
  $bytes = [System.IO.File]::ReadAllBytes($f)
  if ($bytes.Length -ge 3 -and $bytes[0] -eq 0xEF -and $bytes[1] -eq 0xBB -and $bytes[2] -eq 0xBF) {
    $bytes = $bytes[3..($bytes.Length - 1)]
  }
  $text = [System.Text.Encoding]::UTF8.GetString($bytes) -replace "`r", ""
  $text = (($text -split "`n") | ForEach-Object { $_ -replace '[ \t]+$', '' }) -join "`n"
  $text = $text.TrimEnd("`n") + "`n"
  [System.IO.File]::WriteAllBytes($tmp, [System.Text.Encoding]::UTF8.GetBytes($text))
  if ($git) { $h = (git hash-object $tmp).Trim() }
  else { $h = (Get-FileHash -Algorithm SHA256 -LiteralPath $tmp).Hash.ToLowerInvariant() }
  $lines += "$p`t$h"
}
[System.IO.File]::WriteAllText($Out, (($lines -join "`n") + "`n"), (New-Object System.Text.UTF8Encoding($false)))
Remove-Item -LiteralPath $tmp -ErrorAction SilentlyContinue
```

Canonical means: BOM stripped, CRLF/CR → LF, trailing spaces/tabs stripped per line, exactly
one trailing newline. That makes "hash differs" mean "a human meaningfully edited this file" —
including on Windows checkouts. The canonicalizer is text-only; the template's sole binary
(the logo) sits in the never-write set, so within one run its mangled-but-consistent hashes
still prove equality — never compare a binary's hashes across shells (the resume rule's
recompute already covers this). **Self-test before trusting any classification:** create
`.sdd-update/vector.txt` with exactly these bytes — BOM, then `line one`, two spaces, CRLF,
then `line two`, CRLF (shell: `printf '\357\273\277line one  \r\nline two\r\n'`; PowerShell:
write those bytes with `[System.IO.File]::WriteAllBytes`) — and hash it. Expected: git-blob
`e5c5c5583f49a34e86ce622b59363df99e09d4c6`, or SHA-256
`e9024f1a07d29d52ad3aa5e1a18e94db1f3a9fd32b89e39d47c472cd99071e13`. Any other result → stop;
do not classify with a broken hasher.

**Never-write paths take no matrix verdict** — template-side changes to them are report
lines only, and a Target-only never-write file (e.g. `CHANGELOG.md` when the base predates
it) is *reported as available* under `.sdd-update/target/`, never added. The only exceptions
are the two carve-outs below (`.specs/` template surface, `.memory-bank/` seeds still
byte-equal to Base); Phase 6's re-hash proof covers the never-write set minus files updated
through those carve-outs.

Now classify every remaining path (B/T/P are canonical hashes; the **shape scope filters
every case**, adds included — a path under an absent half is skipped and listed as "skipped
(half absent)"):

| Case | Condition | Verdict |
| --- | --- | --- |
| 1 | B == T | template unchanged — skip |
| 2 | B ≠ T, P == B | AUTO: never touched — take Target (inside the batch gate) |
| 3 | B ≠ T, P == T | DONE — already current (the idempotency case) |
| 4 | B ≠ T, P differs from both | MERGE only if a mechanical three-way merge of the **canonicalized** B/T/P copies (`git merge-file`, i.e. diff3 — never the raw checkouts, whose line endings differ) completes with zero conflict hunks — the only accepted proof of disjointness; any conflict hunk → CONFLICT |
| 5 | in Target only | ADD; a local file already at that name: P == T → DONE (already current), P ≠ T → COLLISION conflict |
| 6 | in Base only | P == B → propose DELETE (asked individually); P ≠ B → conflict, default: keep as user orphan |
| 7 | in Base, absent in project | user deleted it — skip, **never resurrect**; if B ≠ T, list it in the report ("template changed X in <v>, absent here — if you renamed it, merge manually from `.sdd-update/target/`") |

**Special handlers** override the plain matrix:

- `AGENTS.md` (or the renamed constitution) — **section-anchored three-way**, never a
  wholesale write. Managed-settings **values** are the user surface; the template may
  rewrite the prose and comments around them. Split B, T and P at Base's headings; anchor
  ladder: exact heading →
  `formerly:` aliases from the ledger → language-invariant structural signatures (the
  `DocLanguage:` key, the `architecture:` YAML block, the markdown table whose first column
  matches `` `/sdd-*` ``, the bullet run under the preference-capture rule). Four user
  surfaces always carry from P verbatim: managed-settings **values** · the preference
  bullets (a changed template *default* bullet applies only if P's list still equals the old
  default) · the `architecture:` block including its comment line · unknown-name command-table
  rows (row-wise merge by command name; a template row whose body file the project lacks is
  *offered*, never auto-inserted — the table follows the file verdict). User sections with no
  Base anchor stay in place untouched. Anchors unmatchable (translated, restructured, merged
  into a larger constitution) → degrade to a guided section-by-section merge, each change
  proposed individually. Snapshot writes route through `/sdd-architecture-update`'s gate —
  never a second write path.
- `.gitignore` — line-ensure the template's entries; never replace the file.
- `.vscode/settings.json` and `.claude/settings.json` — key-level three-way on template keys
  only; a user-changed value (e.g. re-enabled `autoMemoryEnabled`) is a per-key conflict
  defaulting to keep; user keys are invisible.
- `CLAUDE.md` — ensure the `@AGENTS.md` line exists; keep every user addition.
- Scout twins (`.claude/agents/sdd-scout.md` + `.github/agents/sdd-scout.agent.md`) — replace
  bodies as a pair, preserve user-added frontmatter keys (a pinned `model:`), then verify the
  two bodies are byte-identical; remind the user to re-check pinned model names.
- `.specs/README.md` and `.specs/*/.gitkeep` are template surface (normal matrix); everything
  else under `.specs/` is never-write. `.memory-bank/` seeds still byte-equal to Base are
  case 2; anything else there is never-write.
- **Pinned files** — first line after the frontmatter is
  `<!-- featherspec-pin: user-owned; /sdd-featherspec-update reports but does not propose changes here -->`
  → report template-side changes, never ask, never write.

Ledger entries in `(base, target]` also contribute, in version order: `key-migrations:`
(offered), `slot-edits:` (the only operation allowed inside a user slot — each confirmed
individually), `semantic-flips:` (stated plainly, ratified explicitly), `data-notes:`
(matched against the project's actual documents, listed, **never auto-applied**).

Persist the classification as `.sdd-update/_worklist.md` before presenting the preview: one
row per non-skip path — path, case, verdict, decision, status
(`pending | applied | kept | parked | skipped`), and the canonical hash of what was written
(empty until applied). Case-1 skips and half-absent paths may be summarized as counts.

## Phase 3 — Preview (the single batch gate)

Present one summary in `DocLanguage`, e.g.:

```text
FeatherSpec update: 1.0.0 → 1.3.0   (ledger walk: 1.1.0 · 1.2.0 · 1.3.0)
Shape: dual · Constitution: AGENTS.md · 44 files examined
  14 unchanged · 9 auto-apply (never modified by you) · 3 already current
   6 new files · 1 template removal proposed (asks separately)
   3 section/key merges: AGENTS.md, .gitignore, .vscode/settings.json
   2 conflicts needing your decision · 1 pinned file (template changed it — FYI)
   1 semantic flip to ratify · 1 data migration offered
Backup: branch featherspec-backup/pre-1.3.0, then one commit at the end.
Apply? (yes / details <file> / abort)
```

One yes covers the AUTO/ADD/MERGE batch and the two git writes (backup branch, final
commit). Deletions, each conflict, each slot-edit, each semantic flip and each data
migration keep their individual asks. A ledger rename group gets **one** enumerated
confirmation for the whole group, not one per file. Unusually many case-4 conflicts on
wiring the user says they never touched → offer a re-baseline via the bootstrap scoring
("your files best match a state between v<a> and v<b> — use it as Base?").

## Phase 4 — Rollback point

With git: `git branch featherspec-backup/pre-<target>`. Without git: copy every file about
to be written into `featherspec-backup-<date>/` at the repository root — **outside**
`.sdd-update/`, so cleanup can never delete the only backup. The final report always prints
the exact restore command.

## Phase 5 — Apply

Deterministic order: adds → replaces → merges → resolved conflicts → gated deletions. After
**every** file, tick its `_worklist.md` row *and record the canonical hash of what was
written* in the same change set — that is how a resume recognizes its own output instead of
re-merging merged files. Conflict dialogue, five options:

1. **keep mine** — nothing written; offer the pin comment (placed *after* the frontmatter,
   never above it).
2. **take template** — your copy goes to the backup first.
3. **merge** — draft shown in full before writing.
4. **review manually** — live file untouched; target parked in `.sdd-update/review/`.
5. **rename mine** (collisions) — a **command-level** operation: rename body *and* loader
   together, rewrite the loader's pointer link and `name:`, update the user's table row, and
   only then place the template's files.

Never write conflict markers into live wiring — a stray `<<<<<<<` in a command body poisons
every future session. Data migrations run per document, diff shown, applied only on yes;
skips are reported with their consequence.

## Phase 6 — Validate (mechanical, before anything final)

Check, scoped to files **present in the project or written this run** — never completeness
against the template's catalogue, and a missing before- or after-hash is a **FAIL**, never a
vacuous match: loader ↔ body links per command · the frontmatter triangle, for
template-known commands only — every present body has a description, its loader (in shapes
that have loaders) names and links it, the constitution table carries its row; wording may
differ, existence and linkage may not; user-owned commands are advisory lines, never a
miss · instructions loader ↔ rule link per `.claude/rules` file, each loader's `applyTo`
equal to its rule's `paths:` globs (in shapes that have the Copilot
half) · scout twin bodies byte-identical · `.vscode`
location keys · `.gitignore` template lines (including `.sdd-update/`) · the `@AGENTS.md`
line where the shape has `CLAUDE.md` · the three byte-slots (managed-settings values,
preference bullets, `architecture:` block) byte-equal to Phase 2 **except** bytes changed by
individually-confirmed slot-edits recorded in the worklist · every unknown-name table row
still present · every silently-applied file canonical-hash-equal to the fetched Target ·
the **never-write set re-hashes equal to Phase 2** — the machine-checkable proof that no
project knowledge was lost · no unlisted leftovers in `review/` · every worklist row
terminal. The constitution's ~200-line target is a *report line*, never a gate — user length
must not block an update. Any real miss → fail loudly, keep the state files, offer the
rollback, and **do not write the stamp**.

## Phase 7 — Finalize

1. Write `FeatherSpecVersion: <target>` into the constitution's managed block — **last**.
2. The one sanctioned user-state write, per the constitution's own sync gate: append a
   "Changed Recently" line to `.memory-bank/activeContext.md` (skip if an entry for this
   target already exists) and a `techContext.md` line when the stack facts changed.
3. The pre-approved single commit, staging an **enumerated path list** — never a blanket
   add-everything: `chore: FeatherSpec template <base> -> <target>`.
4. Offer to delete `.sdd-update/` (with git — the branch is the backup; without git, point
   at `featherspec-backup-<date>/` and leave deletion to the user).
5. Report in `DocLanguage`: counts (including unchanged template files that carry local
   edits — an FYI, not an action), per-conflict resolutions, `review/` leftovers, data
   migrations applied/skipped, pinned files with pending template changes, the restore
   command, pinned-model recheck, **"VS Code needs a full restart to discover new prompt
   and instructions files (Claude Code does not)"**, and "run `/sdd-overview` to verify."

Re-running after success is free: stamp == target lands everything in cases 1/3 — zero
writes, and Phase 6 doubles as a health check.

## Advisory mode (Copilot-ejected repositories)

Bodies were merged into `.prompt.md` files and the layout redrawn (see the wiki's
*Committing to One Tool*), so file identity is gone. Say so honestly: stage the Target,
produce a file-by-file guidance report mapping staged bodies onto the restructured files via
that page's conversion tables — fetching the page is one more command under the Phase 1
network gate (`https://github.com/GregorBiswanger/featherspec/wiki/Committing-to-One-Tool`);
unreachable → map staged bodies onto the merged `.prompt.md` files by command name, propose
placements section by section, and say the tables were unavailable. Apply changes together,
file by file, on request. *Never destroys, usually automatic* — here: never destroys,
guided.

## Appendix — Version ledger (append-only; one entry per release)

Machine-followable migration knowledge that plain diffs cannot carry. Fields, all optional:
`adds:` · `renames:` · `deletes:` · `key-migrations:` · `section-renames:` (with
`formerly:`) · `slot-edits:` · `data-notes:` (with detect/offer) · `semantic-flips:` ·
`probes:`. File *content* merges endpoint-to-endpoint in one three-way; these *semantic*
entries compose sequentially across skipped versions.

### 0.1.0 — initial (2026-08-03)
- adds: the template — 8 skill-based workflows under `.claude/skills/`, rules, Memory Bank
  seeds, spec lifecycle, Copilot persona

### 0.2.0 — minor (2026-08-03)
- adds: adaptive product-owner interview in `/sdd-specify`

### 0.3.0 — major, pre-1.0 (2026-08-03)
- renames: `.claude/skills/<name>/SKILL.md` → `.claude/commands/<name>.md` (8 workflows)
- adds: `.github/prompts/*.prompt.md` thin loaders · `.claude/rules/plans.md` · persisted
  `NNNN-slug.plan.md` plan files
- key-migrations: `.vscode/settings.json` — replace `chat.agentSkillsLocations` with
  `chat.promptFilesLocations`
- data-notes: pre-0.3 plans lived in chat only (detect: an active spec without a `.plan.md`
  sibling · offer: backfill a plan file from the spec)
- probes: `.claude/skills/` exists ⇒ 0.1–0.2 · `.claude/commands/` without
  `sdd-clarify.md` ⇒ 0.3

### 0.4.0 — minor (2026-08-03)
- adds: `/sdd-clarify` (body + loader) · five criterion shapes · Ask-first tier and fast
  path in the constitution
- data-notes: plan steps gain `Verified:` fields, the traceability Test column and the state
  vocabulary open|built|verified (detect: a plan step without a `Verified:` line · offer:
  per-file upgrade)
- probes: `sdd-clarify.md` present, constitution without a *Progress & state sync*
  section ⇒ 0.4

### 1.0.0 — stabilization (2026-08-04)
- adds: always-loaded *Progress & state sync (gate)* section · `projectbrief.md` wired into
  specify/compile
- semantic-flips: `/sdd-architecture-update` now runs unprompted on drift (previously:
  propose and wait)
- probes: sync gate present, no `sdd-architecture-scan.md` ⇒ 1.0.0

### 1.1.0 — minor (2026-08-26)
- adds: `/sdd-architecture-scan` (body + loader) · `sdd-scout` agents (both tools) ·
  `.claude/rules/architecture-map.md` · `.architecture/` + `.sdd-scan/` locations ·
  `.gitignore` line `.sdd-scan/`
- slot-edits: snapshot header comment gains `· last deep scan: never` (confirm individually)
- key-migrations: `.vscode/settings.json` `chat.agentFilesLocations` — `.claude/agents`
  entry set to `false`
- probes: `sdd-architecture-scan.md` present, no `FeatherSpecVersion:` line ⇒ 1.1.0

### 1.2.0 — minor (2026-08-26)
- adds: `/sdd-featherspec-update` (body + loader) · `FeatherSpecVersion:` stamp in the
  managed settings block · `.gitignore` line `.sdd-update/` · `CHANGELOG.md` (adoption-time
  file — never written into an existing project; read it under `.sdd-update/target/`)
- section-renames: "Repository Settings (managed by /sdd-setup)" → "Repository Settings
  (managed by /sdd-setup and /sdd-featherspec-update)" (formerly: the /sdd-setup-only
  heading)
- slot-edits: managed block gains the `FeatherSpecVersion:` line (written by Phase 7, exempt
  from slot rules) · the `DocLanguage:` line's comment is condensed to one line (value
  untouched)

### 1.3.0 — minor (2026-08-27)
- adds: architect persona, delegated research grounding, quality-gates line and spec
  backlink in the plan workflow · reactivation move (done → active) in `/sdd-lifecycle`
- data-notes: specs gain a `**Plan:**` line under Status (detect: spec without one · offer:
  insert, linking an existing plan file) · plans gain a `**Quality gates:**` line in
  Approach (detect: plan without one · offer: per-file insert)
- probes: `sdd-plan.md` without the architect persona ⇒ ≤1.2.0

### 1.4.0 — minor (2026-08-27)
- adds: `/sdd-clean` (body + loader) — context cleanup for the persistent markdown with a
  token report · `/sdd-architecture-update` may recommend it on visible Memory Bank growth
- probes: `sdd-clean.md` absent ⇒ ≤1.3.0
