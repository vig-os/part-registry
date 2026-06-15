---
name: Schema change (this instance)
about: Change this registry's own data shape — columns, formats, or docs/SCHEMA.md
title: "feat(<scope>): "
labels: ["schema-change"]
---

<!--
Use this when you evolve THIS instance's own schema: a column, format, enum, or
rule in your own docs/SCHEMA.md and CSVs. Record changes use registry-update.md.
Set the title type to match the work (feat / fix / chore) and the scope to one of
schema, docs, repo, templates.

Note: the read-only `template/` mirror is the upstream contract, refreshed only by
template-sync — never hand-edit it here. Diverging your own docs/SCHEMA.md from
that mirror is exactly what this template is for; just be aware that the wider
your divergence, the more manual each future upstream sync becomes.
-->

## Summary

<!-- What structural change is being made to this instance, and the motivation. -->

## Change details

- [ ] Column added / renamed / removed
- [ ] Format or enum changed
- [ ] Issue / PR templates changed
- [ ] Repo configuration / docs changed

## Data migration

<!-- How do this registry's existing rows migrate to the new shape? Note any
     break to the data shape and how registry.csv / print_log.csv were updated. -->

## Divergence from upstream

<!-- Does this move your schema away from the upstream contract (template/SCHEMA.md)?
     Note it so future template-sync PRs are easier to reconcile. Write "n/a" if not. -->

## Checklist

- [ ] `docs/SCHEMA.md` updated to match the new structure.
- [ ] Existing data (`registry.csv` / `print_log.csv`) migrated to the new shape, or a migration note provided above.
- [ ] Column order / append-only / sort invariants are preserved or the change is documented.
- [ ] Backward/forward-compatibility considered (new columns appended at the end).

## Linked issue

Closes #
