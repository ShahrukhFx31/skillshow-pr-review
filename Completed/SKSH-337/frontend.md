# Frontend PR Review — skillshow-admin-ui (`SKSH-337`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `a79de419`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-16 (`a79de419`, backend `e03c28f`)  
**Scope:** Crew dashboard "Recent Assigned Events" — searchable/sortable server list, pagination bar contract, realtime invalidation (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 2 (0 Critical, 0 High) — **0 Open**, **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/api/services/crewDashboardService.ts` | `getCrewAssignedEvents` with search/sort params |
| `src/api/types/crew-dashboard.types.ts` | Assigned-events DTO + query param type |
| `src/constants/queryKeys.ts` | `crewAssignedEventsQueryKey` |
| `src/pages/dashboard/crew/dashboard/components/assigned-events-*` | Section, table, columns, responsive cards |
| `src/pages/dashboard/crew/dashboard/hooks/use-crew-assigned-events.ts` | Query + sort/search/pagination wiring |
| `src/pages/dashboard/crew/dashboard/hooks/use-crew-assigned-events-table-controls.ts` | Debounced search + server sort state |
| `src/pages/dashboard/crew/dashboard/hooks/use-crew-list-pagination-bar.ts` | Shared hidden pagination + bar state |
| `src/pages/dashboard/crew/dashboard/utils/crew-assigned-events-table.utils.ts` | Sort key mapping helpers |
| `src/utils/edit-request-realtime.utils.ts` | Invalidate/refetch assigned-events queries |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `useCrewAssignedEvents` owns query + controls + pagination contract | ✅ |
| `useCrewListPaginationBar` centralizes `hidden`/`bar` shape | ✅ DRY |
| `applyServerSort` wired for assigned-events table sorting | ✅ Contract |
| Query key includes page, size, search, sortBy, sortOrder | ✅ |
| Realtime invalidation includes `crewAssignedEventsQueryKey` | ✅ |
| Protected pagination modules untouched | ✅ |

### Positive notes

- **Server-list consistency:** assigned-events now follows the same server list structure used elsewhere (controls hook + hidden pagination + `PaginationBar`).
- **UX:** header search input is debounced, page resets correctly on filter/sort changes via controls hook.
- **Cross-stack alignment:** frontend now supports backend search/sort contract for assigned events.

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

**Re-review evidence:** section consumes `bar`/`hidden` from `useCrewAssignedEvents`, and table receives those directly instead of re-creating pagination state in-place.

---

---
PaginationBar page sizes exceed backend crew limit cap

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-table.tsx  
Lines: 1-70

**Re-review evidence:** backend now aligns to 100 max limit; table pagination is bound to server list controls and no longer has the prior 25-cap mismatch behavior.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Assigned events pagination bypasses usePagination bar/hidden | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-section.tsx | 16-50 | — |
| 2 | PaginationBar page sizes exceed backend crew limit cap | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-table.tsx | 1-70 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High/Medium frontend blockers.
