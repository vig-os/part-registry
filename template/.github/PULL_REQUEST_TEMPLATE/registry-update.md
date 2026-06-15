---
name: Registry data update
about: Add, edit, or void records in registry.csv / append to print_log.csv
title: "record(registry): "
labels: ["record"]
---

## Summary

<!-- Which parts/records change and why. -->

## Records affected

| ID | Action | Status before -> after |
|----|--------|------------------------|
|    | mint / bind / void / edit |  |

## Checklist

- [ ] `registry.csv` is still sorted by `id` ascending.
- [ ] IDs match the 14-char alphabet (no `0 O 1 I L`).
- [ ] Per-status fields are valid (see `docs/SCHEMA.md`): `bound`/`retired` rows carry `bound_at` + `bound_by`; `unbound`/`void` rows do not.
- [ ] `labeled=yes` appears only on `bound` or `retired` rows.
- [ ] Status transitions are allowed (`unbound -> bound`, `bound -> bound`, `bound -> retired`, `* -> void`; no back-transitions).
- [ ] `print_log.csv` changes are append-only (no edits/deletes of existing rows).

## Linked issue

Closes #
