# Release cycle

How the **upstream** `vig-os/part-registry` template cuts a release. The released
artifact is the **schema + templates that forks consume** — there is no build, no
binary, no image of our own. A release is: a dated `CHANGELOG.md`, a refreshed
[`template/`](../template/) contract mirror, a bumped schema version stamp, a git
tag, and a GitHub Release.

> Forks don't use this. A fork keeps only `main` and makes record changes
> ([`CONTRIBUTING.md`](../CONTRIBUTING.md)); it has no `dev`/`release` cycle
> unless it chooses to version its own schema, in which case this is the model.

This is a lighter, image-free adaptation of the org's
[`devcontainer` release cycle](https://github.com/vig-os/devcontainer/blob/main/docs/RELEASE_CYCLE.md).
The changelog mechanics reuse that repo's `prepare-changelog` tool (single
source) — the release workflows run it from inside the
`ghcr.io/vig-os/devcontainer` image rather than reimplementing it.

## SemVer for the schema

Versions describe the **schema/template shape forks consume**, not the data.
Following [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html):

| Bump | Meaning | Examples |
|------|---------|----------|
| **MAJOR** (`X.0.0`) | Breaking — a fork's existing rows or parser stop being valid. | Remove / rename / reorder a column; remove an enum value; change a status/transition rule; change `components`/`properties` cell encoding. |
| **MINOR** (`X.Y.0`) | Backward-compatible addition. | Append a new column at the end; add an optional enum value; add a new issue/PR template. |
| **PATCH** (`X.Y.Z`) | No shape change. | Docs/template wording, clarifications, typo fixes. |

**Pre-1.0 caveat:** while at `0.y.z`, the public API is not yet stable, so a
breaking change bumps the **MINOR** (`0.1.z → 0.2.0`) and additions bump the
**PATCH**. (That is why the column redesign in this cycle is `0.2.0`, not `1.0.0`.)

Every breaking change must spell out fork-migration steps in `CHANGELOG.md` and
the release PR — the downstream [`template-sync`](../.github/workflows/template-sync.yml)
flags a changed `template/SCHEMA.md` as *potentially breaking*, but it never
migrates a fork's data.

## Branching

```
dev  ──cut──▶  release/X.Y.Z  ──PR──▶  main  ──(tag X.Y.Z + GitHub Release)
 ▲                                       │
 └──────────────  sync-main-to-dev  ◀────┘
```

`main` and `dev` are protected; nothing is pushed to them directly. Structure
work accumulates on `dev` under `## Unreleased`; a release freezes that into a
dated section, ships it to `main`, tags it, and syncs `main` back into `dev`.

## The flow

Three manual `workflow_dispatch` workflows plus one human gate. All commits to
protected branches are GitHub-signed via `vig-os/commit-action`.

### 1. Prepare — [`prepare-release.yml`](../.github/workflows/prepare-release.yml)

Dispatch with the target `version` (e.g. `0.2.0`). It:

- validates the version, that `release/X.Y.Z` and the tag don't exist, and that
  `## Unreleased` has entries;
- **freezes** the changelog on `dev`: `## Unreleased` content → `## [X.Y.Z] - TBD`,
  leaving a fresh empty `## Unreleased` for ongoing work;
- cuts `release/X.Y.Z` from that commit and strips the empty `## Unreleased` from
  the release branch;
- opens a **draft** PR `release/X.Y.Z → main`.

Run with **dry-run** first to validate without side effects.

### 2. Review — the draft PR

Use the [`release.md`](../.github/PULL_REQUEST_TEMPLATE/release.md) checklist.
Verify the changelog, fork-migration notes for any breaking change, and that the
version matches the SemVer rule above. Fix issues with `bugfix/<n>-…` PRs into
`release/X.Y.Z`. When satisfied, **mark the PR ready for review** and get it
approved. Don't tag yet.

### 3. Finalize — [`finalize-release.yml`](../.github/workflows/finalize-release.yml)

Dispatch with the same `version`. It checks the PR is ready + approved and the
tag is absent, then on the release branch:

- sets the date: `## [X.Y.Z] - TBD` → `## [X.Y.Z](…/releases/tag/X.Y.Z) - YYYY-MM-DD`;
- bumps the **schema version stamp** in [`docs/SCHEMA.md`](SCHEMA.md) and
  [`.github/.template-sync-version`](../.github/.template-sync-version) to `X.Y.Z`;
- refreshes the [`template/`](../template/) contract mirror from the live files;
- commits all of that to the release branch;
- creates and pushes the annotated tag `X.Y.Z`;
- creates a **draft** GitHub Release with the changelog section as notes.

### 4. Promote — human gate

We publish **draft-first**: the workflow only *creates* the draft Release, a
human publishes it. With GitHub immutable releases enabled, **publishing locks
the tag and assets**, so this is the deliberate point of no return.

1. **Publish** the draft GitHub Release from the Releases UI.
2. **Merge** the approved `release/X.Y.Z → main` PR.

The merge to `main` triggers
[`sync-main-to-dev.yml`](../.github/workflows/sync-main-to-dev.yml), which opens a
PR bringing the dated `## [X.Y.Z]` section and the fresh `## Unreleased` back into
`dev` (auto-merge when clean; labelled `merge-conflict` with instructions when
not).

## The devcontainer image pin

`prepare-release.yml` / `finalize-release.yml` run the changelog tool inside
`ghcr.io/vig-os/devcontainer:<tag>`. The tag is pinned in each workflow's
*Resolve image tag* step (an optional `.vig-os` `DEVCONTAINER_VERSION=` overrides
it). Bump it in lockstep with the `devcontainer` release whose `prepare-changelog`
you want.

## Quick reference

```bash
# 1. Prepare (dry-run first, then for real)
gh workflow run prepare-release.yml  -f version=X.Y.Z -f dry-run=true
gh workflow run prepare-release.yml  -f version=X.Y.Z

# 2. Review the draft PR; mark ready; get approval.

# 3. Finalize (tag + draft Release)
gh workflow run finalize-release.yml -f version=X.Y.Z

# 4. Publish the draft Release in the UI, then merge the release PR.
#    sync-main-to-dev opens the main -> dev PR automatically.
```
