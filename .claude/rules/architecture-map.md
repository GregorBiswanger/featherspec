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

> Note: path-scoped rules load when a matching file is **read**. A brand-new map has
> not been read yet, so `/sdd-architecture-scan` restates the schema and these budgets
> inline — that restatement is labelled and this file stays authoritative.
