# Schema

**Schema version:** 0.2.2

<!-- The version this document describes. The release cycle bumps it (and the
template-sync version marker) when a release is cut — see docs/RELEASE_CYCLE.md.
A fork reads this to tell which schema release it tracks. -->

Hand-maintained documentation of the data files in this registry. There is
intentionally **no machine-readable contract or automated validation in this
repo yet** — those live elsewhere and will be wired in later (see the repo
issues). Until then, keep this file in sync by hand when the structure changes,
and route structural edits through the Feature / Bug / Chore issue forms +
`schema-change` PR template.

## Conventions used everywhere

- **IDs** are 14-character nano-ids over the no-lookalike alphabet
  `23456789ABCDEFGHJKMNPQRSTUVWXYZ` (no `0`, `O`, `1`, `I`, `L`).
  Regex: `^[23456789ABCDEFGHJKMNPQRSTUVWXYZ]{14}$`.
- **Timestamps** are ISO-8601 with offset, e.g. `2026-06-01T14:00:00+00:00`.
- **Actors** are GitHub usernames, recorded bare (e.g. `alice`) — no prefix.
  Changes can only be made through signed commits / PRs, so the actor is always
  a GitHub user by construction.
- CSV files use a header row, `,` delimiter, UTF-8, and `\n` line endings.
  Empty fields are the empty string (no `NULL`).
- **Quoting** follows RFC-4180: a cell is double-quoted only when its value
  embeds a `,`, `"`, or newline — in practice the JSON-encoded `components` /
  `properties` cells (see [Cell encoding](#cell-encoding)). Inner `"` are doubled
  (`""`). Every other cell stays unquoted. An empty structured field is the empty
  string, never `[]` or `{}`.

## `registry.csv`

Canonical part records — the single source of truth. **Sorted by `id`
ascending** so diffs only show the rows that actually changed.

Lifecycle: `mint` (create an `unbound` ID) -> `bind` (attach to a real part,
`status=bound`) -> optionally `label` (stick a physical sticker, `labeled=yes`).
A sticker is only applied to a part that is/was bound, so `labeled=yes` requires
`status` to be `bound` or `retired`. There are two terminal states:
`retired` (the part was genuinely in service and is now decommissioned or
superseded — history is preserved) and `void` (the ID never became a valid
in-service part: mis-mint, scrapped on arrival, mistake).

Column order (13 columns):

| # | Column | Type / format | Notes |
|---|--------|---------------|-------|
| 1 | `id` | id (14-char) | Canonical ID. Unique across the file. |
| 2 | `status` | enum | `unbound` \| `bound` \| `retired` \| `void`. |
| 3 | `minted_at` | datetime | When the ID was minted. |
| 4 | `minted_by` | actor | Who minted the ID. |
| 5 | `bound_at` | datetime | When bound to a part. Set on `bound`/`retired`; empty on `unbound`/`void`. |
| 6 | `bound_by` | actor | Who bound the part. Set on `bound`/`retired`; empty on `unbound`/`void`. |
| 7 | `labeled` | enum | `yes` \| `no` — whether a physical sticker is on the part. `yes` only valid when `bound` or `retired`. |
| 8 | `location` | string | Physical location / path. |
| 9 | `type` | string | Part type, e.g. `PT100`. Drives the expected `properties` schema (later). |
| 10 | `components` | JSON array (CSV-quoted) | Direct subcomponent IDs when this part is an assembly; empty string otherwise. See [Cell encoding](#cell-encoding) and [Component referential integrity](#component-referential-integrity). |
| 11 | `properties` | JSON object (CSV-quoted) | Type-specific attributes — absorbs the former `description`, `vendor`, `part_number`, `notes`. Empty string when none. See [Cell encoding](#cell-encoding). |
| 12 | `last_edited_at` | datetime | Last edit timestamp. |
| 13 | `last_edited_by` | actor | Who last edited the row. |

### Cell encoding

`components` and `properties` are structured fields encoded as **JSON inside a
single CSV cell**:

- **`components`** — a JSON **array** of subcomponent IDs, e.g.
  `["23456789ABCDEF","2345ABCDEFGHJK"]`. Empty assemblies use the empty string,
  not `[]`.
- **`properties`** — a JSON **object** mapping keys to values, e.g.
  `{"vendor":"Acme","part_number":"PT100-A"}`. No attributes uses the empty
  string, not `{}`.

Because JSON embeds `,` and `"`, these cells are **double-quoted per RFC-4180**,
with inner `"` doubled. As they appear in the raw CSV:

```
"[""23456789ABCDEF"",""2345ABCDEFGHJK""]"
"{""vendor"":""Acme"",""part_number"":""PT100-A""}"
```

Any compliant CSV reader (Python `csv`, pandas) decodes these transparently; a
naive line/`split(',')` reader does not.

**`properties` keys.** Keys are free-form strings, but the recommended set
absorbs the columns folded in by the v0.2 redesign: `description`, `vendor`,
`part_number`, `notes`. `retired` and `void` rows record their reason under a
`reason` key (see [Per-status field rules](#per-status-field-rules)). The `type`
field drives the *expected* key set for a part; a formal per-type `properties`
schema is a future refinement and is not enforced here.

### Component referential integrity

When `components` is populated, these rules apply (documented here; **not
enforced** in this repo yet — validation tooling will enforce them later):

- **Exists** — every listed ID MUST exist as an `id` in `registry.csv`.
- **No self-reference** — a part MUST NOT list its own `id`.
- **No cycles** — composition is acyclic; a part MUST NOT be its own ancestor.
- **Bound subcomponents** — each listed component SHOULD be a `bound` or
  `retired` part (a real physical part), not `unbound`/`void`.

### Per-status field rules

- **`unbound`** — minted but not attached. Must NOT carry `bound_at`,
  `bound_by`, `type`, `location`, `components`, or `properties`. `labeled` MUST be
  `no` (a sticker is only applied once a part is bound).
- **`bound`** — attached to a real part. MUST carry `bound_at` and `bound_by`;
  should carry `type`/`location` and type-specific `properties`. `labeled` is
  `yes` when a sticker is physically on the part, or `no` when intentionally
  unlabeled (e.g. a subcomponent, or a part that already carries its own printed
  ID).
- **`retired`** — was bound and in service, now decommissioned or superseded.
  Retains the historical binding fields (`bound_at`, `bound_by`, `type`,
  `location`, `properties`); `labeled` may stay `yes` if the sticker is still on
  the part. Record the reason in `properties`. Terminal.
- **`void`** — the ID never became a valid in-service part (mis-mint, scrapped,
  mistake). MUST NOT carry `bound_at`/`bound_by`; `labeled` is `no`. Record the
  reason in `properties`. Terminal.

### Allowed status transitions

`unbound -> bound` (bind), `bound -> bound` (rebind), `bound -> retired`
(decommission), `* -> void`. No back-transitions (no `bound -> unbound`, no
`retired -> bound`, no `void -> *`). New rows are born `unbound` or `bound`.

## `print_log.csv`

**Append-only** audit trail of label print events — one row per ID per print.
Never edit or delete existing rows.

| # | Column | Type / format | Notes |
|---|--------|---------------|-------|
| 1 | `id` | id (14-char) | The part whose label was printed. |
| 2 | `printed_at` | datetime | When the print happened. |
| 3 | `printed_by` | actor | Who printed it. |
| 4 | `layout` | string | Label layout, e.g. `horz` / `vert` / `flag`. |
| 5 | `size_mm` | number | Short-side size in mm. |
| 6 | `extra` | string | Layout-specific extra (e.g. cable OD). May be empty. |
