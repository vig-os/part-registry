# Changelog

All notable **structure** changes to this repo are documented here — schema,
docs, issue/PR templates, and tooling. Record (data) changes to `registry.csv`
and `print_log.csv` are **not** tracked here: the CSVs and their git history are
the data SSoT, and `print_log.csv` is itself an append-only audit log. See
[`CONTRIBUTING.md`](CONTRIBUTING.md) for the two-axis split.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
for the schema/template shape that forks consume.

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
