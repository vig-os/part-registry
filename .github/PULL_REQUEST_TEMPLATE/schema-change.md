---
name: Structure change
about: Repo development (feature/bug/chore) — columns, formats, docs, templates, or config
title: "feat(<scope>): "
labels: ["schema-change"]
---

<!--
PR template is keyed to *what* changes (data vs structure), not intent. Any
feature/bug/chore touching schema, docs, templates, or tooling uses this one;
record changes use registry-update.md. Set the title type to match the work
(feat / fix / chore) and the scope to one of schema, docs, repo, templates.
-->

## Summary

<!-- What structural change is being made and the motivation. -->

## Change details

- [ ] Column added / renamed / removed
- [ ] Format or enum changed
- [ ] Issue / PR templates changed
- [ ] Repo configuration / docs changed

## Impact on forks

<!-- How must existing forks migrate? Note any breaking change to the data shape. -->

## Changelog Entry

<!-- Paste the exact entry you added to CHANGELOG.md under ## Unreleased.
     If no update is needed (a pure-internal chore with no user-visible or
     workflow impact), write "No changelog needed" and explain why. Record/data
     PRs are exempt — they use registry-update.md and are not tracked here. -->

## Checklist

- [ ] `docs/SCHEMA.md` updated to match the new structure.
- [ ] Existing data (`registry.csv` / `print_log.csv`) migrated to the new shape, or a migration note provided above.
- [ ] Column order / append-only / sort invariants are preserved or the change is documented.
- [ ] Backward/forward-compatibility considered (new columns appended at the end).
- [ ] `CHANGELOG.md` updated under `## Unreleased` (or "No changelog needed" noted above).

## Linked issue

Closes #
