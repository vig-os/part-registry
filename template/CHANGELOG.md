# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0](https://github.com/vig-os/part-registry/releases/tag/0.2.0) - 2026-06-15

### Added

- **Downstream template-sync** ([#9](https://github.com/vig-os/part-registry/issues/9))
  - `template/`: a pristine, machine-managed **upstream reference set** — verbatim
    `SCHEMA.md` / `README.md` / `CHANGELOG.md` plus worked-example
    `registry.csv` / `print_log.csv`. Forks never hand-edit it; refreshed at
    release time from the live files.
  - `.github/workflows/template-sync.yml`: a scheduled workflow, inherited by
    every instance, that detects a new upstream *release* and opens a PR
    refreshing `template/` and the functional machinery per the
    `.github/.template-sync-paths` allowlist. A change to `template/SCHEMA.md`
    is surfaced as a *potentially breaking* schema review; records, `docs/SCHEMA.md`,
    and the repointed `config.yml` are never touched. Strictly release-gated via
    `.github/.template-sync-version`.
  - Commits are created via `vig-os/commit-action` (GitHub-signed) using the
    `COMMIT_APP` / `RELEASE_APP` tokens — no PAT. No-op in the upstream template
    itself.
  - A read-only copy of the worked-example `registry.csv` / `print_log.csv` is
    mirrored under `template/`; fork-setup and release steps documented in
    `CONTRIBUTING.md`, `README.md`, and `CLAUDE.md`.
- **`components` column** ([#8](https://github.com/vig-os/part-registry/issues/8)) —
  models assembly composition as a list of subcomponent registry IDs. Referential
  integrity (exists / no self-reference / no cycles / bound-or-retired) is
  documented in `docs/SCHEMA.md` but not yet auto-enforced.
- **`properties` column** ([#8](https://github.com/vig-os/part-registry/issues/8)) —
  a free-form, type-specific property bag that absorbs the former `description`,
  `vendor`, `part_number`, and `notes` columns.
- **Release cycle** ([#7](https://github.com/vig-os/part-registry/issues/7))
  - `docs/RELEASE_CYCLE.md`: the documented `dev → release/X.Y.Z → main` flow and
    the SemVer-for-schema policy (MAJOR = column removed/renamed/reordered or rule
    change; MINOR = appended column / new template; PATCH = wording), with the
    `0.y` pre-1.0 caveat.
  - `prepare-release.yml` / `finalize-release.yml`: workflows that freeze the
    changelog, cut `release/X.Y.Z`, refresh `template/` + the schema version stamp,
    tag, and create a **draft** GitHub Release. They reuse the `devcontainer`
    image's `prepare-changelog` (single source) and `vig-os/commit-action`.
  - `sync-main-to-dev.yml`: opens a PR syncing `main` back into `dev` after a
    release merges.
  - `.github/PULL_REQUEST_TEMPLATE/release.md`: the release checklist.
  - A **Schema version** stamp in `docs/SCHEMA.md`, bumped at release alongside
    `.github/.template-sync-version`, so a fork can tell which release it tracks.

### Changed

- **CHANGELOG adopts the `prepare-changelog` conventions** ([#7](https://github.com/vig-os/part-registry/issues/7)) —
  generic Keep a Changelog header and inline-link version headings
  (`## [0.1.0](…tag…) - date`), so the release tool maintains this file as a
  single source. The structure-only / two-axis scope note now lives only in
  `CONTRIBUTING.md`.
- **`components` / `properties` cell encoding defined** ([#11](https://github.com/vig-os/part-registry/issues/11)) —
  both structured fields are now JSON inside a single CSV cell: `components` a
  JSON array of IDs, `properties` a JSON object. The cells are double-quoted per
  RFC-4180 (inner `"` doubled); empty stays the empty string. Documented in
  `docs/SCHEMA.md`, demonstrated in the worked-example rows, and reflected in the
  `record.yml` request form.
- **BREAKING: `registry.csv` columns re-shuffled** ([#8](https://github.com/vig-os/part-registry/issues/8))
  into a more logical order (`id, status, minted_at, minted_by, bound_at,
  bound_by, labeled, location, type, components, properties, last_edited_at,
  last_edited_by`); 16 → 13 columns. Forks must re-map their rows.

### Removed

- **BREAKING: `registry.csv` `batch` column** ([#8](https://github.com/vig-os/part-registry/issues/8)) —
  low value and recoverable from `minted_at` / the print log.
- **BREAKING: `registry.csv` `description` / `vendor` / `part_number` / `notes`
  columns** ([#8](https://github.com/vig-os/part-registry/issues/8)) — folded into
  the new `properties` field.
- **BREAKING: `print_log.csv` `batch_label` column** ([#8](https://github.com/vig-os/part-registry/issues/8)) —
  orphaned by the registry `batch` removal; 9 → 8 columns.
- **BREAKING: `print_log.csv` `output_mode` and `copies` columns** ([#8](https://github.com/vig-os/part-registry/issues/8)) —
  print-job mechanics with no audit value; the printed label is reproducible from
  `layout` / `size_mm` / `extra`. 8 → 6 columns.

## [0.1.0](https://github.com/vig-os/part-registry/releases/tag/0.1.0) - 2026-06-11

### Added

- **Initial project scaffolding** ([#1](https://github.com/vig-os/part-registry/issues/1))
  - Registry data model (`registry.csv`, `print_log.csv`) and `docs/SCHEMA.md`
    field/format/lifecycle reference.
  - Issue forms and PR templates encoding the per-axis invariant checklists.
  - Contribution governance (`CONTRIBUTING.md`, `CLAUDE.md`, `README.md`): the
    two-axis workflow and the `main` / `dev` / `release/X.Y.Z` branching model.
  - `CHANGELOG.md` (Keep a Changelog) tracking structure changes only — record
    (data) PRs are exempt — wired into the `schema-change.md` PR template.
- **Fork setup checklist** ([#3](https://github.com/vig-os/part-registry/issues/3))
  - Document the one-time fork steps (delete seed rows, create workflow labels,
    protect `main`, repoint the schema link) in `CONTRIBUTING.md`, linked from
    `README.md`.
  - Flag the upstream-absolute *Schema reference* link in `config.yml` so forks
    repoint it to their own `docs/SCHEMA.md`.
