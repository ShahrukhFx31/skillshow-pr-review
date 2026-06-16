# Frontend PR Review — skillshow-admin-ui (`SKSH-337`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `a79de419`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-12 (`a79de419`)  
**Scope:** Crew dashboard "Recent Assigned Events" — server-paginated/searchable/sortable table, API hook, realtime invalidation (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 2 (0 Critical, 2 High) — **0 Open**, **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/api/services/crewDashboardService.ts` | `getCrewAssignedEvents` with search/sort params |
| `src/api/types/crew-dashboard.types.ts` | Assigned-events DTOs + list params |
| `src/constants/queryKeys.ts` | `crewAssignedEventsQueryKey` |
| `src/pages/dashboard/crew/dashboard/components/assigned-events-*` | Section, table, columns, responsive cards |
| `src/pages/dashboard/crew/dashboard/hooks/use-crew-assigned-events.ts` | Query + pagination bar wiring |
| `src/pages/dashboard/crew/dashboard/hooks/use-crew-assigned-events-table-controls.ts` | Search/sort/pagination controls |
| `src/pages/dashboard/crew/dashboard/hooks/use-crew-list-pagination-bar.ts` | Shared `bar`/`hidden` for server lists |
| `src/pages/dashboard/crew/dashboard/utils/crew-assigned-events-table.utils.ts` | Sort key resolution + column sort order |
| `src/utils/edit-request-realtime.utils.ts` | Invalidate/refetch assigned-events on socket events |
| `src/pages/user/account/general/crew/**` | Performance reviews page size 5 → 10 (tangential) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `useCrewListPaginationBar` centralizes `bar`/`hidden` for server-paginated crew tables | ✅ |
| `applyServerSort` + sortable columns aligned to backend `sortBy` keys | ✅ |
| `filterKey` resets page on search/sort change | ✅ |
| `queryKey` includes page, pageSize, search, sortBy, sortOrder | ✅ |
| `CrewDashboardTruncatedName` / display utils reused across tables | ✅ DRY |
| Realtime utils cover `crewAssignedEventsQueryKey` | ✅ |
| Prior pagination duplication + page-size cap drift | ✅ Fixed (#1, #2) |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **Server-list contract:** `useCrewAssignedEvents` composes table controls + `useCrewListPaginationBar` and passes `bar`/`hidden` into `AssignedEventsTable`; table uses `applyServerSort` and `<PaginationBar {...bar} />`.
- **Search UX:** Debounced search in card `extra` with `filterKey` page reset.
- **Defaults:** `CREW_ASSIGNED_EVENTS_PAGE_SIZE` uses `DEFAULT_LIST_QUERY.pageSize` (10), aligned with backend `LIST_QUERY_PAGINATION.DEFAULT_PAGE_SIZE`.

---

## GitHub comments

_No open findings._

---

## Findings

---
Assigned events pagination bypasses usePagination bar/hidden

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-section.tsx  
Lines: 16-50

**Re-review evidence:** `useCrewAssignedEvents` builds `bar`/`hidden` via `useCrewListPaginationBar` (with API `total`) and passes them to `AssignedEventsTable`, which uses `pagination={hidden}` and `<PaginationBar {...bar} />`.

---

---
PaginationBar page sizes exceed backend crew limit cap

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-table.tsx  
Lines: 48

**Re-review evidence:** Backend now caps crew `limit` at `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100), matching `PAGE_SIZE_OPTIONS` on `PaginationBar`. No silent 25-row clamp.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Assigned events pagination bypasses usePagination bar/hidden | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-section.tsx | 16-50 | — |
| 2 | PaginationBar page sizes exceed backend crew limit cap | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-table.tsx | 48 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High/Medium blockers.
