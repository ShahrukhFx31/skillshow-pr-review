# PR review (SKSH-406) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#340](https://github.com/SkillshowFx/skillshow-admin-ui/pull/340) |
| Branch | `SKSH-406` → `main` |
| Head | `669eda0387e4cf520c51da508a41aba6506bb726` |
| Scope | Admin list total fallback; Edit Request Pagination → PaginationBar |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-15 — head `669eda03` unchanged; sibling totals still invent length |

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx` | **Consumed only** (not modified) ✅ |
| use-pagination, use-server-table-controls, table-sort | **Not modified** ✅ |

### Positive notes

- Migrating Edit Request list chrome to `PaginationBar` matches app-wide rows-per-page UI.
- `EditRequestListPaginationBar` removes duplicated Table/Cards adapters.
- Main admin list total now uses `?? 0` (no longer invents page length).

## GitHub comments

### `src/pages/adminEditRequest/index.tsx`

- **L227** — Assignment-returns / feedback totals still fall back to page row length (HIGH)

## Findings

---
Fabricated list total from current page length

Risk Level: HIGH
File Path: src/pages/adminEditRequest/index.tsx
Lines: 227-229

Description:
**Contract** — Main `listTotal` was updated to `data?.pagination?.total ?? 0`, but assignment-returns and feedback tabs still use `?? assignmentReturnsRows.length` / `?? feedbackRows.length`, inventing a total when `pagination` is missing.

**Re-verify (669eda03, latest):** Partially fixed — main list OK; `assignmentReturnsTotal` and `feedbackTotal` still synthesize from current page length.

Impact:
- Assignment Returns / Feedback `PaginationBar` (and export counts) can understate the real total and hide further pages.

Recommendation:
Use `?? 0` for both sibling totals, same as `listTotal`:

```ts
const assignmentReturnsTotal = assignmentReturnsData?.pagination?.total ?? 0;
const feedbackTotal = feedbacksData?.pagination?.total ?? 0;
```
---

---
Duplicated PaginationBar adapters in Edit Request list UI

Risk Level: MEDIUM
File Path: src/pages/editRequest/components/EditRequestTable.tsx
Lines: 208

Description:
**DRY** — Table and Cards each copied PaginationBar wiring.

**Re-verify (669eda03):** ✅ Fixed — shared `EditRequestListPaginationBar` used by both.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Fabricated list total from current page length | HIGH | Partially fixed | `src/pages/adminEditRequest/index.tsx` | 227-229 |
| 2 | Duplicated PaginationBar adapters in Edit Request list UI | MEDIUM | ✅ Fixed | `src/pages/editRequest/components/EditRequestTable.tsx` | 208 |

**Merge readiness:** Not ready — 1 open High (sibling tab totals still invent page length).
