# Changelog

All notable **structure** changes to this repo are documented here — schema,
docs, issue/PR templates, and tooling. Record (data) changes to `registry.csv`
and `print_log.csv` are **not** tracked here: the CSVs and their git history are
the data SSoT, and `print_log.csv` is itself an append-only audit log. See
[`CONTRIBUTING.md`](CONTRIBUTING.md) for the two-axis split.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
for the schema/template shape that forks consume.

## Unreleased

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
  - Root `registry.csv` / `print_log.csv` now ship **header-only**; document the
    fork-setup and release steps in `CONTRIBUTING.md`, `README.md`, and
    `CLAUDE.md`.

### Changed

- **Root CSVs are header-only starters** — no seed rows to delete on fork; the
  illustrative example now lives under `template/` (a read-only worked example),
  with every column and format documented in `docs/SCHEMA.md`.

## [0.1.0] - 2026-06-11

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

[0.1.0]: https://github.com/vig-os/part-registry/releases/tag/0.1.0
