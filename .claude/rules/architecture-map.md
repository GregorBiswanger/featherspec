---
paths:
  - .architecture/**/*.md
---

# Architecture map writing rules

`.architecture/` holds curated per-module maps, created by `/sdd-architecture-scan`
only when the snapshot cap would otherwise evict navigation facts. The snapshot in
`AGENTS.md` stays the inventory; a map holds one module's inside view.

- Write in `DocLanguage` from `AGENTS.md`. Budget: ≤ 40 lines per map.
- Content, in order: internal pattern as path patterns · entry points · task
  playbooks ("when doing X: files A → B → C; never Y") · deviations · traps.
- Nothing an agent can infer from the code in seconds; path patterns over path lists.
- Keep each map's `# last reconciled:` line current — `/sdd-architecture-update`
  maintains it when structure drifts.

## Scout reports (`.sdd-scan/reports/`)

Raw per-unit reports from `/sdd-architecture-scan`'s Phase B follow the schema, in order:
Purpose (2 sentences) · Entry points (exact paths) · Internal pattern (path patterns) ·
Dependencies in/out (concrete contract or event files) · Deviations from repo
conventions · Traps and frozen zones · Observed decisions — the observation with its evidence
path, any explanation found in a source quoted with that source's path, and the ADR, decision or
requirement documents seen; never an inferred why.

- Budget: a leaf report fits the schema in **≤ 120 lines**. If an honest report cannot,
  that is the split rule firing — a structural note plus proposed child units, never
  harder compression.
- Three nested budgets, three purposes: ≤ 5 return lines protect the orchestrator ·
  ≤ 120 report lines protect the synthesis · ≤ 40 map lines protect the end artifact.
- A pattern's *purpose* is an observation; its *reason* is not in the code. Record where an
  explanation is written, never what it probably is — `.claude/rules/knowledge-records.md` is
  authoritative for what may later become a confirmed reason.

> Note: path-scoped rules load when a matching file is **read**. A brand-new map has
> not been read yet, so `/sdd-architecture-scan` restates the schema and these budgets
> inline — that restatement is labelled and this file stays authoritative.
