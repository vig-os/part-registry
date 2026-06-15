---
name: Release
about: Cut a versioned schema/template release (release/X.Y.Z → main)
title: "chore(repo): release "
labels: ["release"]
---

<!--
For release/X.Y.Z → main PRs only. Usually opened as a DRAFT by
prepare-release.yml; this template is the review + finalize checklist. See
docs/RELEASE_CYCLE.md for the full flow. Apply the `release` label manually
(there is no labeling automation in this repo).
-->

## Release X.Y.Z

<!-- The version and a one-line summary of what this release ships to forks. -->

## Version choice (SemVer for schema)

<!-- Why this is MAJOR / MINOR / PATCH per docs/RELEASE_CYCLE.md. Remember the
0.y pre-1.0 caveat: a breaking change bumps the MINOR while < 1.0. -->

- [ ] Version follows the SemVer-for-schema rule in `docs/RELEASE_CYCLE.md`.

## Fork migration

<!-- For any breaking change, the exact steps a fork takes to re-map its data.
Write "None — no breaking change" if so. -->

## Checklist

- [ ] `## Unreleased` promoted to `## [X.Y.Z] - <date|TBD>` (run via `prepare-release.yml`).
- [ ] `docs/SCHEMA.md` **Schema version** stamp and `.github/.template-sync-version` bumped to `X.Y.Z`.
- [ ] [`template/`](../../template/) contract mirror refreshed from the live files.
- [ ] Breaking changes (if any) documented with fork-migration notes above and in `CHANGELOG.md`.
- [ ] Data invariants unaffected (a release touches structure only, never records).
- [ ] PR marked **ready for review** and approved before running `finalize-release.yml`.
- [ ] Plan: tag `X.Y.Z` + **draft** GitHub Release, publish from the UI, then merge (draft-first).

## Linked issue

Closes #
