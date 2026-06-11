# Schema

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

Column order (16 columns):

| # | Column | Type / format | Notes |
|---|--------|---------------|-------|
| 1 | `id` | id (14-char) | Canonical ID. Unique across the file. |
| 2 | `status` | enum | `unbound` \| `bound` \| `retired` \| `void`. |
| 3 | `minted_at` | datetime | When the ID was minted. |
| 4 | `batch` | string | Mint batch label, e.g. `B-2026-06-01`. |
| 5 | `bound_at` | datetime | When bound to a part. Empty unless `bound`. |
| 6 | `type` | string | Part type, e.g. `PT100`. |
| 7 | `description` | string | Free-text description. |
| 8 | `vendor` | string | Supplier name. |
| 9 | `part_number` | string | Vendor part number. |
| 10 | `location` | string | Physical location / path. |
| 11 | `notes` | string | Free-text notes. |
| 12 | `minted_by` | actor | Who minted the ID. |
| 13 | `bound_by` | actor | Who bound the part. Empty unless `bound`. |
| 14 | `last_edited_at` | datetime | Last edit timestamp. |
| 15 | `last_edited_by` | actor | Who last edited the row. |
| 16 | `labeled` | enum | `yes` \| `no` — whether a physical sticker is on the part. `yes` only valid when `bound` or `retired`. |

### Per-status field rules

- **`unbound`** — minted but not attached. Must NOT carry `bound_at`,
  `bound_by`, `type`, or `location`. `labeled` MUST be `no` (a sticker is only
  applied once a part is bound).
- **`bound`** — attached to a real part. MUST carry `bound_at` and `bound_by`;
  should carry `type`/`description`/`location`. `labeled` is `yes` when a sticker
  is physically on the part, or `no` when intentionally unlabeled (e.g. a
  subcomponent, or a part that already carries its own printed ID).
- **`retired`** — was bound and in service, now decommissioned or superseded.
  Retains the historical binding fields (`bound_at`, `bound_by`, `type`,
  `description`, `location`); `labeled` may stay `yes` if the sticker is still on
  the part. Record the reason in `notes`. Terminal.
- **`void`** — the ID never became a valid in-service part (mis-mint, scrapped,
  mistake). MUST NOT carry `bound_at`/`bound_by`; `labeled` is `no`. Record the
  reason in `notes`. Terminal.

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
| 7 | `copies` | number | Number of copies printed. |
| 8 | `output_mode` | string | e.g. `single` / `sheet` / `strip`. |
| 9 | `batch_label` | string | Originating batch, if any. |
