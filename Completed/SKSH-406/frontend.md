# PR review (SKSH-406) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#340](https://github.com/SkillshowFx/skillshow-admin-ui/pull/340) |
| Branch | `SKSH-406` → `main` |
| Head | `fda17dbe25dfa24204923173939f73481f66d33e` |
| Scope | Admin list total fallback; Edit Request Pagination → PaginationBar |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Verified | 2026-07-17 vs prior head `669eda03` |

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx` | **Consumed only** (not modified) ✅ |
| use-pagination, use-server-table-controls, table-sort | **Not modified** ✅ |

### Positive notes

- Migrating Edit Request list chrome to `PaginationBar` matches app-wide rows-per-page UI.
- `EditRequestListPaginationBar` removes duplicated Table/Cards adapters.
- All admin list totals (`listTotal`, `assignmentReturnsTotal`, `feedbackTotal`) use `?? 0`.

## GitHub comments

No open findings to post (prior High resolved on head).

## Findings

---
Fabricated list total from current page length

Risk Level: HIGH
File Path: src/pages/adminEditRequest/index.tsx
Lines: 227-229

Description:
**Contract** — Assignment-returns and feedback tabs used `?? rows.length`, inventing a total when `pagination` was missing.

Impact:
- PaginationBar / export counts could understate the real total

Recommendation:
Use `?? 0` for both sibling totals, same as `listTotal`.

Status: ✅ Fixed — `assignmentReturnsTotal` and `feedbackTotal` now use `?? 0` (L258–260 on head).
---

---
Duplicated PaginationBar adapters in Edit Request list UI

Risk Level: MEDIUM
File Path: src/pages/editRequest/components/EditRequestTable.tsx
Lines: 208

Description:
**DRY** — Table and Cards each copied PaginationBar wiring.

Status: ✅ Fixed — shared `EditRequestListPaginationBar` used by both.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Fabricated list total from current page length | HIGH | ✅ Fixed | `src/pages/adminEditRequest/index.tsx` | 227-229 |
| 2 | Duplicated PaginationBar adapters in Edit Request list UI | MEDIUM | ✅ Fixed | `src/pages/editRequest/components/EditRequestTable.tsx` | 208 |

**Merge readiness:** Ready — prior High fully fixed on `fda17dbe`; no new Critical/High/Medium open.
