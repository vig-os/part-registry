# Contributing

This repo is the SSoT for the registry. `main` is **write-protected** — all
changes land via Pull Request. Branching uses `main` / `dev` / `release/X.Y.Z`
with one shortcut for data: record updates go straight to `main`, while structure
work flows through `dev`. See [*Branching model*](#branching-model).

## Fork setup

This is a GitHub **template** — generate your own registry with *Use this
template*, then make it yours (one time):

1. **Delete the seed rows** in [`registry.csv`](registry.csv) and
   [`print_log.csv`](print_log.csv) — they are an illustrative worked example.
   Keep the header row; your data goes underneath.
   [`docs/SCHEMA.md`](docs/SCHEMA.md) documents every column and its format, and a
   read-only copy of the example is mirrored under
   [`template/`](template/) (the upstream contract mirror — see step 6; don't
   edit it).
2. **Install the fork-facing machinery.** Generating from the template copies the
   *upstream's own* `.github/`, which is aimed at developing the template. A fork
   doesn't develop the template, so install the fork-facing set shipped under
   [`template/.github/`](template/.github/) — the `template-sync` workflow, its
   allowlist and version marker, the record/bug/chore forms, `registry-update.md`,
   a fork-scoped `schema-change.md`, a `chore.md` for repo-plumbing work, a
   dispatcher `pull_request_template.md`, and a default `config.yml` — and remove
   the upstream-only development machinery:

   ```sh
   # install the fork-facing machinery into the live .github/
   cp -r template/.github/. .github/
   # remove the upstream-only development machinery
   rm -f .github/ISSUE_TEMPLATE/feature.yml \
         .github/PULL_REQUEST_TEMPLATE/release.md \
         .github/workflows/prepare-release.yml \
         .github/workflows/finalize-release.yml \
         .github/workflows/sync-main-to-dev.yml
   ```

   (Do this before the next step — the copy installs the default `config.yml`.
   This is the one-time bootstrap that activates the sync; thereafter
   **template-sync** keeps the mapped machinery current for you. `config.yml` is
   shipped as a default but is **never** overwritten by sync, and
   `.template-sync-version` is written by the workflow — both are yours.)
3. **Repoint the schema link** in
   [`.github/ISSUE_TEMPLATE/config.yml`](.github/ISSUE_TEMPLATE/config.yml): the
   *Schema reference* contact link is an absolute URL to the upstream repo —
   change it to your fork's own `docs/SCHEMA.md`. (The *App bugs* link can stay;
   the viewer/editor app is a shared upstream project, not forked.)
4. **Create the workflow labels** so the issue forms can apply them: `record`,
   `chore` (plus `schema-change`, which a schema PR carries). `bug` ships with
   every repo. (A fork's synced forms are record/bug/chore — no `feature`; that
   one is upstream-only.) Missing labels don't block issue creation — the label
   is just silently dropped.
5. **Protect `main`**: require a PR + review before merge. A fork keeps **only
   `main`** (record updates branch off it and PR straight back) — there is no
   `dev`/`release` cycle unless you also version your own schema.
6. **Enable template sync** (optional, recommended). The
   [`template-sync`](.github/workflows/template-sync.yml) workflow watches for
   new upstream *releases* and opens a PR that refreshes the upstream **contract
   mirror** under [`template/`](template/), the shared sync plumbing, and the
   fork-facing issue/PR templates — which it **maps** from `template/.github/`
   into your live `.github/`, strictly per
   [`.github/.template-sync-paths`](.github/.template-sync-paths). It never
   touches your records, your [`docs/SCHEMA.md`](docs/SCHEMA.md), README,
   CHANGELOG, or your repointed `config.yml`.
   - **`template/` is a read-only mirror** — don't hand-edit it. When the sync PR
     changes [`template/SCHEMA.md`](template/SCHEMA.md) it flags a *potentially
     breaking* schema change; you then update your own `docs/SCHEMA.md` and
     `registry.csv` on your own terms.
   - It commits through the org GitHub Apps so commits stay **signed**, so it
     needs the `COMMIT_APP_CLIENT_ID` / `COMMIT_APP_PRIVATE_KEY` and
     `RELEASE_APP_CLIENT_ID` / `RELEASE_APP_PRIVATE_KEY` secrets: instances
     inside the `vig-os` org inherit them automatically; external forks install
     equivalent Apps or just delete the workflow.
   - **Existing** instances (generated before this shipped) run the step-2
     bootstrap once by hand — copy `template/.github/.` into `.github/` and pull
     in the `template/` directory — since template inheritance only applies at
     generation time.

Then start minting.

## Workflow

1. **Open an issue** using the right form:
   - *Registry change request* — add/bind/void/edit records (the only path for
     record changes, including in forks).
   - *Feature* / *Bug* / *Chore* — develop the repo itself: columns, formats,
     docs, templates, or tooling.
2. **Branch off the right base** (see [*Branching model*](#branching-model)):
   record updates branch off `main`; structure work branches off `dev`.
3. **Make the change** and keep [`docs/SCHEMA.md`](docs/SCHEMA.md) accurate.
4. **Open a PR** with the matching template and `Closes #<issue>`: record PRs
   target `main`; structure PRs target `dev`.

```mermaid
flowchart LR
  main[main] -->|branch| rec["registry/&lt;summary&gt;"]
  rec -->|"PR (record)"| main
  dev[dev] -->|branch| topic["feature/N-&lt;summary&gt;"]
  topic -->|"PR (structure)"| dev
  dev -->|cut| rel["release/X.Y.Z"]
  rel -->|"PR (release)"| main
```

## Change types

Two axes. *What* you touch (the **artifact**) picks the PR template and its
checklist; *intent* (feature/bug/chore) picks the issue form. There are three
artifacts: records, the schema (data shape), and repo plumbing.

| What changes | Issue form | PR template | PR label |
|--------------|-----------|-------------|----------|
| Records | Registry change request | `registry-update.md` | `record` |
| Schema (data shape / `docs/SCHEMA.md`) | Feature / Bug / Chore | `schema-change.md` | `schema-change` |
| Repo plumbing (CI, deps, docs, templates, config) | Chore | `chore.md` | `chore` |

A feature, a bug fix, and a chore **on the schema** all use `schema-change.md` —
they share the same invariant checklist. A chore that touches neither the records
nor the data shape (an action-pin bump, a README edit, a CI tweak) uses
`chore.md` instead, which carries no schema checklist. The issue form additionally
tags intent with `feature` / `bug` / `chore`.

The issue forms apply their label automatically. PR templates *suggest* the
artifact label (apply it manually, since there is no labeling automation in this
repo yet). A fresh fork must create these labels first — see
[*Fork setup*](#fork-setup).

## Branching model

A `main` / `dev` / `release/X.Y.Z` hierarchy mapped onto the two axes — the
**data axis goes straight to `main`**, the **structure axis flows through `dev`**:

| Branch | Base | Target | Purpose |
|--------|------|--------|---------|
| `main` | — | — | Protected. Production registry + released schema/templates. |
| `dev` | — | `main` (via `release/*`) | Protected integration branch for **structure** work. |
| `release/X.Y.Z` | `dev` | `main` | Cut a versioned schema/template release. Protected. |
| `feature/<issue#>-<summary>` | `dev` | `dev` | New schema/tooling feature. |
| `bugfix/<issue#>-<summary>` | `dev` or `release/X.Y.Z` | same | Structure bug fix. |
| `chore/<summary>` | `dev` | `dev` | Maintenance, no issue required. |
| `registry/<summary>` | `main` | `main` | **Record updates** — straight to `main`, no `dev`. |

**Forks skip `dev` and `release/*` entirely.** A fork is a data store, not a
versioned product: it keeps only `main` plus `registry/<summary>` record
branches that PR directly into `main`. The `dev`/`release` cycle exists only in
the upstream template, where the schema and templates are versioned for forks to
consume.

## Naming conventions

Follows [Conventional Commits](https://www.conventionalcommits.org/), adapted:

- **Commits / PR titles / issue titles**: `type(scope): subject` — imperative,
  lowercase, no trailing period.
  - Types — split by axis (see *Change types* above):
    - **Data axis** (record changes): `record`. One type covers every action
      (mint/bind/retire/void/edit) — the action lives in the subject, not the
      type. Pairs with the `registry` / `print-log` scopes.
    - **Structure axis** (repo development): `feat`, `fix`, `docs`, `chore`,
      `ci`, `refactor`. Pairs with the `schema` / `docs` / `repo` / `templates`
      scopes.
  - Scopes: `registry`, `print-log`, `schema`, `docs`, `repo`, `templates`.
  - Examples: `record(registry): bind 3 sensors on loop-A`,
    `record(print-log): append label print for 23456789ABCDEF`,
    `feat(schema): add labeled column`.
- **Branches** — the prefix is the branch type, **not** the commit type
  (branch `feature/…`, commit `feat(scope): …`). See
  [*Branching model*](#branching-model):
  - Structure: `feature|bugfix/<issue#>-<kebab-summary>` off `dev`
    (e.g. `feature/12-add-labeled-column`); `chore/<summary>` needs no issue.
  - Releases: `release/X.Y.Z` off `dev` (e.g. `release/1.2.0`).
  - Routine data updates: `registry/<summary>` off `main`
    (e.g. `registry/B-2026-06-08-bind`).

## Changelog

[`CHANGELOG.md`](CHANGELOG.md) tracks **structure** changes only — the
development axis (schema, docs, templates, tooling). It uses the
[Keep a Changelog](https://keepachangelog.com/) format with an `## Unreleased`
section.

- **Structure PRs** (`schema-change.md` and `chore.md`, targeting `dev`): add an
  entry under `## Unreleased` using the right category (`Added` / `Changed` /
  `Deprecated` / `Removed` / `Fixed` / `Security`) and paste it into the PR's
  *Changelog Entry* section. Skip only pure-internal chores with no user-visible or
  workflow impact (note "No changelog needed" and why). Format:
  `- **Bold title** ([#issue](url))` with indented detail sub-bullets.
- **Record PRs** (`registry-update.md`, targeting `main`) are **exempt** — the
  CSVs and their git history are the data SSoT and `print_log.csv` is already an
  append-only audit log, so a changelog would duplicate them.
- **On `release/X.Y.Z`** the accumulated `## Unreleased` entries are promoted to
  a dated `## [X.Y.Z]` section; don't add new `## Unreleased` items there.
- Never edit entries below `## Unreleased` (released, dated sections).

## Releasing (upstream only)

The full release cycle — `dev → release/X.Y.Z → main`, SemVer-for-schema, and the
`prepare-release` / `finalize-release` / `sync-main-to-dev` workflows — lives in
[`docs/RELEASE_CYCLE.md`](docs/RELEASE_CYCLE.md). In short: a release freezes the
`## Unreleased` changelog into a dated `## [X.Y.Z]`, regenerates the
[`template/`](template/) contract mirror and the schema version stamp /
[`template/.github/.template-sync-version`](template/.github/.template-sync-version)
from the live files, tags `X.Y.Z`, and ships a draft GitHub Release a human
publishes.

The `template/` **contract mirror** (`SCHEMA.md`, `README.md`, `CHANGELOG.md`,
`registry.csv`, `print_log.csv`) is a generated snapshot (like a lockfile): the
live files are authored; the mirror is derived — never hand-edit it. Forks'
`template-sync` reads the tag's mirror to detect what changed since their last
sync, so a stale mirror would mislead every downstream. The **fork-facing
templates** under [`template/.github/`](template/.github/) are the exception —
they are hand-authored source (the templates forks actually run), not generated,
and are edited directly through the normal structure-change flow.

## Data invariants

When editing the CSVs (by hand or via the app), preserve:

- `registry.csv` **sorted by `id` ascending**.
- `print_log.csv` **append-only** (never edit/delete existing rows).
- Per-status field rules and allowed transitions (`unbound -> bound`,
  `bound -> bound`, `bound -> retired`, `* -> void`; no back-transitions) — see
  `docs/SCHEMA.md`.
- `labeled=yes` only on `bound` or `retired` rows (a sticker only exists on a
  part that is/was bound).
- New columns are appended at the end (forward-compatible).
