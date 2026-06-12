# Frontend PR Review — skillshow-admin-ui (`SKSH-337`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `6baff488`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-11 (`6baff488` — feedback: server-table contract, pagination bar/hidden, limit cap)  
**Scope:** Crew dashboard assigned-events section (server search/sort/pagination), shared list-pagination hook extraction, edit-request table UX polish (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 4 (1 Critical, 1 High, 2 Medium) — **1 Critical Open**, **1 High Open**, **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx` | **Modified** — optional `pageSizeOptions` prop (see #1) |
| `use-pagination.ts` | **Modified** — delegates to `useListPaginationBar` (see #1) |
| `use-server-table-controls.ts` | **Modified** — `initialPageSize` param (see #1) |
| `table-sort.ts` | **Modified** — `getServerColumnSortOrder` helper (see #1) |
| `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | Not modified |

### Files reviewed

| File | Change |
|------|--------|
| `src/hooks/use-list-pagination-bar.ts` | **New** shared `bar`/`hidden` for server lists |
| `src/hooks/use-pagination.ts` | Refactor → `useListPaginationBar` |
| `src/hooks/use-server-table-controls.ts` | `initialPageSize` support |
| `src/components/pagination-bar.tsx` | Optional `pageSizeOptions` |
| `src/utils/table-sort.ts` | `getServerColumnSortOrder` |
| `src/pages/adminEditRequest/hooks/use-admin-edit-request-list-pagination.ts` | Thin wrapper → `useListPaginationBar` |
| `src/pages/dashboard/crew/dashboard/**` | Assigned-events section, table, columns, hook |
| `src/utils/edit-request-realtime.utils.ts` | Invalidate/refetch assigned-events |
| `src/pages/user/account/general/crew/**` | Performance reviews page size 5 → 10 (tangential) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `useCrewAssignedEvents` uses `useServerTableControls` + `applyServerSort` + `useListPaginationBar` | ✅ Fixed (prior #1) |
| `queryKey` includes page, pageSize, search, sortBy, sortOrder | ✅ |
| Backend limit cap now 100 — `PaginationBar` options aligned | ✅ Fixed (prior #2) |
| `useListPaginationBar` extracted; `use-pagination` + admin edit-request hook migrated | ✅ DRY |
| Four frozen table/pagination modules modified in same PR | ❌ Protected scope — see #1 |
| `CREW_ASSIGNED_EVENTS_PAGE_SIZE` = 5 (`DEFAULT_LIST_PAGINATION`) vs backend default 10 | ❌ Contract — see #2 |
| Default sort `createdAt` not shown on any column | ⚠️ UX — see #3 (Medium) |

### Positive notes

- **Contract compliance:** Assigned events now follow the frozen server-table stack (`useServerTableControls`, `applyServerSort`, `useListPaginationBar`, `PaginationBar`) — addresses initial-review feedback.
- **Search/sort:** Debounced search in card header; server sort wired through column `key` + `getServerColumnSortOrder`.
- **Realtime:** Socket handlers invalidate/refetch `crewAssignedEventsQueryKey`.
- **UX polish:** Shared truncation helpers for edit-request title/name columns; tag/status wrapping on mobile cards.

---

## GitHub comments

### 1. `src/hooks/use-pagination.ts` line 3

**PR comment (line 3):** **Critical (Protected module):** This PR modifies four frozen table/pagination modules (`pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`). Changes look backward-compatible, but protected-module edits need explicit ticket scope or a dedicated infra PR. Please confirm SKSH-337 authorizes these shared changes, or split the hook extractions into a separate review.

### 2. `src/pages/dashboard/crew/dashboard/constants.ts` line 9

**PR comment (line 9):** **High (Contract):** `CREW_ASSIGNED_EVENTS_PAGE_SIZE` uses `DEFAULT_LIST_PAGINATION.pageSize` (5) while backend `LIST_QUERY_PAGINATION.DEFAULT_PAGE_SIZE` and frontend `DEFAULT_LIST_QUERY.pageSize` are both 10. Use `DEFAULT_LIST_QUERY.pageSize` (or `10`) for the initial page size so crew assigned-events matches the shared list contract.

---

## Findings

---
Protected table/pagination modules modified

Risk Level: CRITICAL  
**Status:** Open  
File Path: skillshow-admin-ui/src/hooks/use-pagination.ts  
Lines: 1-36

Description:
**Protected module.** This PR modifies four frozen shared modules to support server-side assigned events:

| File | Change |
|------|--------|
| `pagination-bar.tsx` | Optional `pageSizeOptions` prop |
| `use-pagination.ts` | Delegates `bar`/`hidden` to new `useListPaginationBar` |
| `use-server-table-controls.ts` | `initialPageSize` parameter |
| `table-sort.ts` | `getServerColumnSortOrder` export |

Changes are additive/backward-compatible (defaults preserve prior behavior), and `use-admin-edit-request-list-pagination.ts` correctly migrates to `useListPaginationBar`. However, SKSH-337 does not explicitly scope shared infrastructure changes.

Impact:
- Shared pagination/sort behavior changes ride on a feature ticket — higher regression risk for all server lists using these hooks.
- Future list pages depend on hooks modified outside a dedicated infra review.

Recommendation:
Either (a) confirm SKSH-337 explicitly authorizes these protected-module updates and mark **Accepted**, or (b) split `useListPaginationBar`, `getServerColumnSortOrder`, and hook refactors into a small prerequisite PR reviewed on its own, then rebase the assigned-events UI on top.

**PR comment (`use-pagination.ts` line 3):** **Critical (Protected module):** This PR modifies four frozen table/pagination modules (`pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`). Changes look backward-compatible, but protected-module edits need explicit ticket scope or a dedicated infra PR. Please confirm SKSH-337 authorizes these shared changes, or split the hook extractions into a separate review.

---

---
Assigned events initial page size out of sync with list contract

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/dashboard/crew/dashboard/constants.ts  
Lines: 9

Description:
**Contract.** `CREW_ASSIGNED_EVENTS_PAGE_SIZE` is set from `DEFAULT_LIST_PAGINATION.pageSize` (`PAGE_SIZE_OPTIONS[0]` = **5**), while:

- Backend `LIST_QUERY_PAGINATION.DEFAULT_PAGE_SIZE` = **10**
- Frontend `DEFAULT_LIST_QUERY.pageSize` = **10**

`useCrewAssignedEvents` passes `initialPageSize: CREW_ASSIGNED_EVENTS_PAGE_SIZE` into `useServerTableControls`, so the table loads with 5 rows per page on first visit — inconsistent with the documented cross-stack default of 10.

```9:9:skillshow-admin-ui/src/pages/dashboard/crew/dashboard/constants.ts
export const CREW_ASSIGNED_EVENTS_PAGE_SIZE = DEFAULT_LIST_PAGINATION.pageSize;
```

Impact:
- Crew dashboard assigned-events table shows 5 rows while other admin lists default to 10.
- If `limit` were omitted, backend would return 10 rows while UI state expects 5.

Recommendation:
```typescript
export const CREW_ASSIGNED_EVENTS_PAGE_SIZE = DEFAULT_LIST_QUERY.pageSize;
```

**PR comment (`constants.ts` line 9):** **High (Contract):** `CREW_ASSIGNED_EVENTS_PAGE_SIZE` uses `DEFAULT_LIST_PAGINATION.pageSize` (5) while backend and `DEFAULT_LIST_QUERY` default to 10. Use `DEFAULT_LIST_QUERY.pageSize` for the initial page size.

---

---
Assigned events pagination bypasses usePagination bar/hidden

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/dashboard/crew/dashboard/hooks/use-crew-assigned-events.ts  
Lines: 18-51

Description:
**Contract / DRY.** Initial review flagged hand-rolled hidden pagination in `AssignedEventsTable` and `usePagination` called with `total: 0`.

Impact:
- Duplicated pagination markup; partial hook usage.

Recommendation:
Use `useServerTableControls` + `useListPaginationBar` and pass `bar`/`hidden` to the table.

**Re-review evidence:** `useCrewAssignedEvents` now composes `useServerTableControls`, `useListPaginationBar`, and passes `bar`/`hidden` through `AssignedEventsSection` → `AssignedEventsTable`. Table uses `applyServerSort` and `pagination={hidden}`.

---

---
PaginationBar page sizes exceed backend crew limit cap

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/components/pagination-bar.tsx  
Lines: 11-18

Description:
**Contract (cross-stack).** Initial review flagged `PaginationBar` options up to 100 while backend service capped `limit` at 25.

Impact:
- UI page size could exceed rows returned.

Recommendation:
Align backend cap with validation or restrict `pageSizeOptions`.

**Re-review evidence:** Backend `parseCrewPaginatedListQuery` now caps at `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100), matching `PAGE_SIZE_OPTIONS`. Optional `pageSizeOptions` prop added for future per-table restriction if needed.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Protected table/pagination modules modified | CRITICAL | Open | skillshow-admin-ui/src/hooks/use-pagination.ts | 1-36 | 3 |
| 2 | Assigned events initial page size out of sync with list contract | HIGH | Open | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/constants.ts | 9 | 9 |
| 3 | Assigned events pagination bypasses usePagination bar/hidden | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/hooks/use-crew-assigned-events.ts | 18-51 | — |
| 4 | PaginationBar page sizes exceed backend crew limit cap | HIGH | ✅ Fixed | skillshow-admin-ui/src/components/pagination-bar.tsx | 11-18 | — |

**Merge readiness:** **Not merge-ready** — 1 open Critical (protected-module scope) + 1 open High (default page size 5 vs 10). Backend has no open blockers. Mark finding #1 **Accepted** if ticket scope includes shared hook changes; fix #2 before merge.
