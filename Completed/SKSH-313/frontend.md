# Frontend PR Review — skillshow-admin-ui (`SKSH-313`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-313`  
**Base:** `main...HEAD` @ `7e7080a0`  
**Re-verified:** 2026-06-08  
**Scope:** Event view crew management (list/add/remove), related sub-tables (athletes/crew/videos), event ref routing (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 6 (1 High, 4 Medium, 1 Accepted-as-High) — **0 Open**, **2 Accepted**, **4 Fixed**

### Protected modules

No changes to `use-server-table-controls.ts`, `pagination-bar.tsx`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, or `destructive-action-confirm-modal.tsx`.

---

## GitHub comments (Open findings)

*None — all findings Fixed or Accepted.*

---

---
Unrelated dashboard card subtitles removed (scope / UX regression)

Risk Level: HIGH  
File Path: src/pages/management/app-users/dashboard/index.tsx  
Lines: 101-109 (removed)  
(+ crew-users, skillshow-users, partners, events dashboard — same pattern)

Description:
Five dashboard pages drop their `Card` `title` blocks while this ticket is event-view work.

**Status:** Accepted — intentional dashboard title cleanup per team decision.

---

---
Athletes/crew tabs fetch unbounded pages via `fetchAllEventRelatedList`

Risk Level: HIGH  
File Path: src/pages/events/utils/event.utils.ts  
Lines: 218-244  
File Path: src/pages/events/onboarding/hooks/use-event-view-related-lists.ts  
Lines: 50-60

Description:
**Contract / performance / Global consistency.** Athletes and crew use client-side search/sort/pagination: `fetchAllEventRelatedList` loads page 1, then `Promise.all` for every remaining page at `EVENTS_LIST_MAX_LIMIT` (25) with **no total cap**. Results are filtered/sorted/paginated in `event-view-related-table.tsx`.

Impact:
- An event with 500 registered athletes triggers ~20 parallel list API calls on tab open and holds the full dataset in memory.
- Server list endpoints (sort, search, pagination) built in this ticket are unused for athletes/crew; behavior diverges from the videos tab (server-driven).

Recommendation:
Use server-driven lists for athletes/crew, or cap `fetchAllEventRelatedList` (max pages/rows) with server-pagination fallback when total exceeds the cap.

**PR comment (line 219):**  
**High:** `fetchAllEventRelatedList` fans out unbounded parallel page requests (25 rows/page) for athletes/crew — please use server pagination for large lists or add a hard fetch cap with server fallback.

**Status:** Accepted — intentional client-side search/sort/pagination for athletes/crew per team decision; typical event registration counts expected to stay within practical bounds. Revisit if large-event performance becomes an issue.

---

---
Videos tab omits controlled column `sortOrder` (Global consistency)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-columns.tsx  
Lines: 27-75  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-table.tsx  
Lines: 46, 84-94

Description:
**Global consistency:** Videos tab uses server sort via `applyServerSort` + `useServerTableControls`.

**Re-verification (@ `cf062d9d` / `7e7080a0`):** `buildEventViewVideoColumns(sortBy, sortOrder)` adds `columnSortOrder` per column (video-library pattern); section passes `videoSortBy` / `videoSortOrder` into `EventViewRelatedTable`.

**Status:** ✅ Fixed

---

---
Event view tables omit controlled column `sortOrder` (athletes/crew)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/constants.ts  
Lines: 134-166

Description:
Athletes/crew use client-side `sorter: (a, b) => localeCompare(...)` comparators; Ant Table manages sort indicators natively.

**Status:** ✅ Fixed

---

---
Add Crew modal duplicates Add Athlete modal (DRY)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-add-related-modal.tsx  
Lines: 1-78

Description:
Shared `EventViewAddRelatedModal` + `useEventViewAddRelatedModal`; athlete/crew modals are thin wrappers.

**Status:** ✅ Fixed

---

---
Backend crew filters exist; event view filter UI still placeholder

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-actions.tsx  
Lines: 1-22

Description:
Non-functional filter dropdown removed; search-only toolbar.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unrelated dashboard card subtitles removed | HIGH | Accepted | src/pages/management/app-users/dashboard/index.tsx | 101-109 |
| 2 | Athletes/crew `fetchAllEventRelatedList` unbounded fan-out | HIGH | Accepted | src/pages/events/utils/event.utils.ts | 218-244 |
| 3 | Videos tab omits controlled column `sortOrder` | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-view/event-view-related-columns.tsx | 27-75 |
| 4 | Athletes/crew controlled server `sortOrder` (superseded) | MEDIUM | ✅ Fixed | src/pages/events/onboarding/constants.ts | 134-166 |
| 5 | Add Crew modal duplicates Add Athlete modal | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-view/event-view-add-related-modal.tsx | 1-78 |
| 6 | Filter UI placeholder | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-view/event-view-related-actions.tsx | 1-22 |

### Re-review notes (2026-06-08 @ `7e7080a0`)

| Change | Verdict |
|--------|---------|
| `cf062d9d` `buildEventViewVideoColumns` + `columnSortOrder` | **Fixed** videos `sortOrder` finding |
| `fetchAllEventRelatedList` client-side fetch | **Accepted** per team |
| Dashboard titles | **Accepted** |

**Positive notes:** Videos tab uses `useServerTableControls` + controlled column sort + `PaginationBar`; athletes/crew use `usePagination` `hidden`/`bar`; crew CRUD + shared add modal; `DestructiveActionConfirmModal` for removes.

**Merge readiness:** **No open Critical/High/Medium blockers** on frontend. Backend has no open blockers — ticket review complete.
