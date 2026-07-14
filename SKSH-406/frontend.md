# PR review (SKSH-406) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#340](https://github.com/SkillshowFx/skillshow-admin-ui/pull/340) |
| Branch | `SKSH-406` → `main` |
| Head | `cf842b130609bd6efd738b1b3e4fb5cb12efb63b` |
| Scope | Admin list total fallback; Edit Request Pagination → PaginationBar |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-14 — head unchanged `cf842b13`; findings still Open |

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx` | **Consumed only** (not modified) ✅ |
| use-pagination, use-server-table-controls, table-sort | **Not modified** ✅ |

### Positive notes

- Migrating Edit Request list chrome to `PaginationBar` matches app-wide rows-per-page UI.
- Showing the bar when `total > 0` (not only `total > limit`) aligns with `PaginationBar` behavior.

## GitHub comments

### `src/pages/adminEditRequest/index.tsx`

- **L222** — Fabricated list total from current page length (HIGH)

### `src/pages/editRequest/components/EditRequestTable.tsx`

- **L208** — Duplicated PaginationBar adapters in Edit Request list UI (MEDIUM)

## Findings

---
Fabricated list total from current page length

Risk Level: HIGH
File Path: src/pages/adminEditRequest/index.tsx
Lines: 222

Description:
**Contract** — Totals change from `data?.pagination.total ?? 0` to `data?.pagination?.total ?? rows.length`. Falling back to **current page `rows.length`** invents a total when the list envelope omits `pagination`.

Impact:
- `PaginationBar` may show a wrong total (e.g. 10 when the real total is larger), under-paging or hiding further pages.
- Export / tab totals derived from the same values can report a misleading count.

Recommendation:
Keep optional chaining, but use `?? 0` — do not substitute row length for server total. If `pagination` is missing, fix the API contract.
---

---
Duplicated PaginationBar adapters in Edit Request list UI

Risk Level: MEDIUM
File Path: src/pages/editRequest/components/EditRequestTable.tsx
Lines: 208

Description:
**DRY** / **Contract** — `EditRequestTable` and `EditRequestCards` each copy the same `PaginationBar` wiring (`setPage` → `onPageChange(next, limit)`, `setPageSize` → `onPageChange(1, next)`).

Impact:
- Future pagination behavior changes must be edited twice and can drift.

Recommendation:
Lift a single adapter (parent builds `bar`, or a small shared footer) and reuse in both Cards and Table.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Fabricated list total from current page length | HIGH | Open | `src/pages/adminEditRequest/index.tsx` | 222 |
| 2 | Duplicated PaginationBar adapters in Edit Request list UI | MEDIUM | Open | `src/pages/editRequest/components/EditRequestTable.tsx` | 208 |

**Merge readiness:** Not ready — 1 open High finding.
