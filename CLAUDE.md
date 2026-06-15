# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`part-registry` is a **data SSoT**, not an application. It is a GitHub *template*
repo holding two plain-CSV files plus the docs and GitHub forms/templates that
govern how they change. There is **no build, no test suite, no CI, and no code**
here — validation tooling and the viewer/editor app (`vig-os/part-registry-app`)
live in separate repos. "Working in this repo" means editing CSV rows by hand
while preserving invariants, or editing the schema/docs/templates.

The live root CSVs ship with an **illustrative worked example** (a fork deletes
the seed rows and keeps the header). Every column and its format is documented in
`docs/SCHEMA.md`, and `template/registry.csv` / `template/print_log.csv` hold a
read-only copy of the same example (part of the upstream mirror).

## Files

| File | What it is |
|------|-----------|
| `registry.csv` | Canonical part records, **sorted by `id` ascending**. 13 columns. Ships with a worked example; forks delete the seed rows. |
| `print_log.csv` | **Append-only** label-print audit trail. 6 columns. Ships with a worked example; forks delete the seed rows. |
| `docs/SCHEMA.md` | Hand-maintained field/format/lifecycle reference — the authority for all rules below. A fork may trim it to the subset it uses. |
| `template/` (contract mirror) | **Upstream reference set** — pristine, machine-managed copies of `SCHEMA.md`, `README.md`, `CHANGELOG.md` (verbatim) plus worked-example `registry.csv` / `print_log.csv` (rows; their headers are the schema's column fingerprint). Forks **never hand-edit** these; the `template-sync` workflow overwrites them from each release, and a change to `template/SCHEMA.md` is the breaking-change signal. In upstream they are a generated snapshot, refreshed at release time (see `CONTRIBUTING.md` → *Releasing*). |
| `template/.github/` (fork-facing machinery) | **Hand-authored** — everything a fork actually runs: the `template-sync.yml` workflow, the `.template-sync-paths` allowlist, the `.template-sync-version` seed, the record/bug/chore issue forms, `registry-update.md`, a fork-scoped `schema-change.md`, a record-only `pull_request_template.md`, and a default `config.yml`. `template-sync` **maps** these into a fork's live `.github/` (per the `src -> dest` lines in the allowlist); `config.yml` ships as a default but is excluded from sync, and the workflow writes `.template-sync-version` itself. Unlike the contract mirror these are authored, not generated. |
| `.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE/` | This repo's **own** issue forms (registry change request + feature/bug/chore) and PR templates, aimed at **developing the upstream template**. Not synced to forks — forks get the separate set under `template/.github/`. The upstream's `.github/` carries no sync plumbing. |
| `.github/workflows/` (upstream-only) | The release machinery — `prepare-release.yml`, `finalize-release.yml`, `sync-main-to-dev.yml`. Runs only in upstream; forks delete these (they have no `dev`/release cycle). The `template-sync.yml` workflow lives under `template/.github/`, not here. |

## The model (read `docs/SCHEMA.md` before editing data)

A part is a 14-char nano-id tracked through `mint -> bind -> (label)`:

- **IDs**: regex `^[23456789ABCDEFGHJKMNPQRSTUVWXYZ]{14}$` — no-lookalike
  alphabet, excludes `0 O 1 I L`. Unique across `registry.csv`.
- **`status`**: `unbound` -> `bound`, plus terminals `retired` (was in service,
  now decommissioned/superseded) and `void` (never a valid part: mis-mint,
  scrapped, mistake).
- **Timestamps**: ISO-8601 with offset (`2026-06-01T14:00:00+00:00`).
- **Actors** (`minted_by`, `bound_by`, etc.): bare GitHub usernames, no prefix.
- Empty fields are the empty string, never `NULL`.

### Invariants that MUST hold on every data edit

These are the things that are easy to break and have no automated guard in this
repo — verify them by hand:

1. `registry.csv` stays **sorted by `id` ascending** (keeps diffs minimal).
2. `print_log.csv` is **append-only** — never edit or delete existing rows.
3. Allowed status transitions only: `unbound -> bound`, `bound -> bound`
   (rebind), `bound -> retired`, `* -> void`. **No back-transitions.**
4. Per-status field rules:
   - `unbound` — no `bound_at`/`bound_by`/`type`/`location`/`components`/`properties`; `labeled=no`.
   - `bound` — MUST have `bound_at` + `bound_by`; `labeled` is `yes`/`no`.
   - `retired` — retains historical binding fields; record reason in `properties`.
   - `void` — no `bound_at`/`bound_by`; `labeled=no`; record reason in `properties`.
5. `labeled=yes` only on `bound` or `retired` rows.
6. New columns are normally **appended at the end** (forward-compatible) and the
   order is otherwise stable — a deliberate reorder is reserved for breaking
   schema redesigns at a release boundary (e.g. the v0.2 column re-shuffle).
7. `components` references are valid: each listed ID exists, with no
   self-reference or cycles (documented in `docs/SCHEMA.md`; not auto-enforced).

The `registry-update.md` PR template encodes these as a checklist — use it as
your verification list when changing records.

## Making changes — issue -> branch -> PR (no direct pushes to `main`)

`main` is protected. Changes split on **two axes**: *what* is touched (data vs
structure) drives the PR template, label, and invariant checklist; *intent*
(feature/bug/chore) drives which issue form a contributor opens.

| Issue form | Use for | PR template | Label | Commit scope |
|-----------|---------|-------------|-------|--------------|
| "Registry change request" | record data (mint/bind/retire/void/edit) — **the only path for record changes, incl. in forks** | `registry-update.md` | `record` | `registry`, `print-log` |
| "Feature" / "Bug" / "Chore" | repo structure & development (columns/formats, docs, templates, tooling) | `schema-change.md` | `schema-change` (+ `feature`/`bug`/`chore`) | `schema`, `docs`, `repo`, `templates` |

The PR templates stay artifact-based on purpose: a feature, a bug fix, and a
chore on the schema all use `schema-change.md` because they share the same
invariant checklist. Intent lives in the issue and the commit/PR type
(`feat`/`fix`/`chore`).

Branching: `main` (protected) ← `release/X.Y.Z` ← `dev` (protected integration)
← topic branches — with one shortcut: the **data axis skips `dev`**. Record
updates branch off `main` and PR
straight into `main`; structure work branches off `dev` and PRs into `dev`,
reaching `main` only through a `release/X.Y.Z` branch. **Forks keep only `main`**
(record updates only; no `dev`/release cycle). Full table in `CONTRIBUTING.md`.

Workflow: open the matching issue → branch off the right base (`main` for
records, `dev` for structure) → commit referencing the issue → PR into that base
with `Closes #<issue>` and the matching template.

- **Branch naming** (prefix ≠ commit type): structure work uses
  `feature|bugfix/<issue#>-<kebab-summary>` off `dev`
  (e.g. `feature/12-add-labeled-column`), `chore/<summary>` (no issue);
  releases use `release/X.Y.Z` off `dev`; routine data updates use
  `registry/<summary>` off `main` (e.g. `registry/B-2026-06-08-bind`).
- **Commits**: Conventional Commits adapted for this repo — `type(scope): subject`,
  imperative, lowercase, no trailing period. The type follows the same two-axis
  split: **data** changes use `record` (one type for all of
  mint/bind/retire/void/edit — the action is in the subject), **structure**
  changes use `feat fix docs chore ci refactor`. Scopes are listed in the table
  above. (See `CONTRIBUTING.md`.)
- Any **structure change must update `docs/SCHEMA.md` in the same PR** — the
  schema doc is the only contract, so it cannot drift from the data shape.
- **Changelog** — structure PRs add an entry to `CHANGELOG.md` under
  `## Unreleased` (Keep a Changelog categories); skip only pure-internal chores.
  **Record/data PRs do not touch the changelog** — the CSVs and their git
  history are the audit trail. See `CONTRIBUTING.md`.

> Note: the labeling is manual — issue forms auto-apply their label, but PR
> templates only *suggest* it. There is no labeling automation in this repo.

## Relationship to the workspace

This repo's `CONTRIBUTING.md` commit/branch conventions are the operative ones
here; they are a lighter-weight adaptation of the org-wide rules in the parent
`vigOS/CLAUDE.md` (e.g. this repo uses `Closes #<issue>` in the PR, not a
mandatory `Refs:` commit trailer). When they differ, follow this repo's docs.