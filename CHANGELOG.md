# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

### Fixed

- **`template-sync.yml` existing-PR check fails for lack of `pull-requests` scope**
  ([#22](https://github.com/vig-os/part-registry/issues/22)) — the *Check for an
  existing sync PR* step runs `gh pr list` (which queries
  `repository.pullRequests` via GraphQL) with the default `GITHUB_TOKEN`, but the
  top-level `permissions` block granted only `contents: read`, so every instance
  run on a new upstream release aborted with *Resource not accessible by
  integration*. Adds `pull-requests: read` to the workflow's `permissions`. Lands
  upstream so the fix propagates via the next sync (the file is in the allowlist,
  so instances cannot durably patch it themselves).

## [0.2.1](https://github.com/vig-os/part-registry/releases/tag/0.2.1) - 2026-06-15

### Changed

- **GitHub App auth via `client-id`** ([#16](https://github.com/vig-os/part-registry/issues/16)) —
  all auth-using workflows (`template-sync.yml`, `prepare-release.yml`,
  `finalize-release.yml`, `sync-main-to-dev.yml`) bump
  `actions/create-github-app-token` to `v3.2.0` and switch from the deprecated
  `app-id` input to `client-id` (`COMMIT_APP_CLIENT_ID` /
  `RELEASE_APP_CLIENT_ID`). Requires the new `*_CLIENT_ID` org secrets.
- **Fork-facing machinery split out under `template/.github/`** ([#16](https://github.com/vig-os/part-registry/issues/16)) —
  everything a fork runs is now authored under `template/.github/`: the
  `template-sync` workflow, its allowlist and version marker, the record/bug/chore
  issue forms, `registry-update.md`, a fork-scoped `schema-change.md` (for an
  instance's own `docs/SCHEMA.md`), a record-only `pull_request_template.md` with
  a relative `/compare/...` URL, and a default `config.yml`. The allowlist gains a
  `src -> dest` mapping syntax that installs these into a fork's live `.github/`,
  leaving the upstream's own `.github/` aimed solely at developing the template
  (its issue/PR forms keep fork-aware, development-scoped wording, and the release
  workflows stay upstream-only). Instances no longer receive `feature.yml`.
  `config.yml` ships as a default but is never overwritten; `.template-sync-version`
  is written by the workflow and now bumped under `template/.github/` at release.
  Adds a third PR template, `chore.md`, for repo-plumbing work that touches neither
  records nor the data shape (CI/action-pin bumps, deps, README/docs, template
  tidy-ups, config) — closing the gap where the `chore` issue form had no matching
  PR template and such work was forced onto `schema-change.md` with an all-n/a
  checklist. The allowlist maps it into a fork's live `.github/` alongside the other
  PR templates.

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
