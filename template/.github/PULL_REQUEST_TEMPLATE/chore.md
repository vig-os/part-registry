---
name: Repo chore (this instance)
about: Repo plumbing — CI/action pins, deps, README/docs, templates, config (no data, no schema)
title: "chore(<scope>): "
labels: ["chore"]
---

<!--
Use this for upkeep in this registry instance that touches NEITHER records NOR
the data shape (docs/SCHEMA.md): CI/workflow/action pins, dependency bumps,
README/CONTRIBUTING edits, issue/PR template tidy-ups, repo config. A chore that
DOES change docs/SCHEMA.md or the CSV shape uses schema-change.md (it needs the
invariant checklist); record changes use registry-update.md. Set the title type to
match the work (chore / ci / docs / build) and the scope to one of repo, docs,
templates.

Note: the read-only `template/` mirror is the upstream contract, refreshed only by
template-sync — never hand-edit it here.
-->

## Summary

<!-- What upkeep is being done and why. -->

## Change details

- [ ] CI / workflow / action pins / dependency bump
- [ ] README / docs (not docs/SCHEMA.md)
- [ ] Issue / PR templates
- [ ] Repo configuration / meta

## No data or schema impact

- [ ] No change to `registry.csv` / `print_log.csv` or `docs/SCHEMA.md` (data shape). If this box can't be ticked, use `schema-change.md` or `registry-update.md` instead.

## Changelog Entry

<!-- Paste the exact entry you added to CHANGELOG.md under ## Unreleased.
     If no update is needed (a pure-internal chore with no user-visible or
     workflow impact), write "No changelog needed" and explain why. -->

## Linked issue

Closes #
