---
paths:
  - .specs/**/*.md
---

# Specs writing rules

- Write specs in the language specified by `DocLanguage` in `AGENTS.md`
  (default English until `/sdd-setup` sets it).
- Keep acceptance criteria explicit and testable.
- Keep a `**Status:** ...` line near the top (`Draft | In Progress | Implemented | Deprecated`).
- A spec exists in exactly one lifecycle folder (`backlog/`, `active/`, `done/`) at a time.

> Note: path-scoped rules load when a matching file is **read**. When `/sdd-specify` creates
> a brand-new spec, this rule may not be loaded yet — which is why `/sdd-specify` restates
> the essentials (DocLanguage, the `**Status:**` line, the section skeleton) inline.
