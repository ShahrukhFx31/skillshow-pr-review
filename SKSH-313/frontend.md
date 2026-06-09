# Frontend PR Review — skillshow-admin-ui (`SKSH-313`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-313`  
**Base:** `main...HEAD` @ `fb6d13d1`  
**Re-verified:** 2026-06-08  
**Scope:** Event view crew management (list/add/remove), related sub-tables (athletes/crew/videos), event ref routing (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 6 (1 High, 4 Medium, 1 Accepted-as-High) — **2 Open**, **1 Accepted**, **3 Fixed**

### Protected modules

No changes to `use-server-table-controls.ts`, `pagination-bar.tsx`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, or `destructive-action-confirm-modal.tsx`.

---

## GitHub comments (Open findings)

### 1. `src/pages/events/utils/event.utils.ts` line 219

**High:** Athletes/crew tabs now call `fetchAllEventRelatedList`, which fans out parallel requests for every page (`EVENTS_LIST_MAX_LIMIT` 25) with no upper bound — please keep server-driven pagination/search for large lists or cap total rows fetched.

### 2. `src/pages/events/onboarding/components/event-view/event-view-related-table.tsx` line 78

**Medium (Global consistency):** Videos tab still uses server `applyServerSort` but column defs lack controlled `sortOrder` — mirror `video-library-columns.tsx` so headers reflect active server sort.

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
**Contract / performance / Global consistency.** Commit `432529b2` moves athletes and crew from server-driven sort/search/pagination (built earlier in this PR) to client-side handling: `fetchAllEventRelatedList` loads page 1, then `Promise.all` for every remaining page at `EVENTS_LIST_MAX_LIMIT` (25) with **no total cap**. Results are filtered/sorted/paginated in `event-view-related-table.tsx` via `matchesSearch`, column comparators, and `usePagination`.

Impact:
- An event with 500 registered athletes triggers ~20 parallel list API calls on tab open and holds the full dataset in memory.
- Server list endpoints (sort, search, pagination) built in this ticket are unused for athletes/crew; behavior diverges from the videos tab (still server-driven via `useServerTableControls`).
- Risk of slow tab loads and browser memory pressure on large events.

Recommendation:
Either (a) keep server-driven lists for athletes/crew using `useServerTableControls` + debounced search/sort params (consistent with videos and backend contract), or (b) if client-side is intentional for small lists only, cap `fetchAllEventRelatedList` (max pages or max rows) and document the limit; fall back to server pagination when total exceeds the cap.

**PR comment (line 219):**  
**High:** `fetchAllEventRelatedList` fans out unbounded parallel page requests (25 rows/page) for athletes/crew — please use server pagination for large lists or add a hard fetch cap with server fallback.

**Status:** Open

---

---
Videos tab omits controlled column `sortOrder` (Global consistency)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-table.tsx  
Lines: 78-88  
File Path: src/pages/events/onboarding/constants.ts  
Lines: 168-190

Description:
**Global consistency:** The videos tab still uses server sort (`applyServerSort` + `useServerTableControls`) but `EVENT_VIEW_VIDEO_COLUMNS` only set `sorter: true` without per-column `sortOrder`. Athletes/crew tabs no longer apply — they use client-side comparator `sorter` functions (Ant shows sort indicators natively).

Impact:
- Mapped videos: server sort works but column headers do not reflect active `videoSortBy`/`videoSortOrder` after load or tab switch.

Recommendation:
Pass `videoSortBy` / `videoSortOrder` into video column builders and set `sortOrder` via a `columnSortOrder` helper (same as `video-library-columns.tsx`).

**PR comment (line 78):**  
**Medium:** Videos tab server sort is wired, but columns still lack controlled `sortOrder` — pass sort state into column defs like video library.

**Status:** Open

---

---
Event view tables omit controlled column `sortOrder` (athletes/crew)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/constants.ts  
Lines: 134-166

Description:
**Superseded @ `432529b2`:** Athletes/crew columns now use client-side `sorter: (a, b) => localeCompare(...)` comparators; Ant Table manages sort indicators without server `sortOrder` binding.

**Status:** ✅ Fixed (by design change to client-side sort)

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
| 2 | Athletes/crew `fetchAllEventRelatedList` unbounded fan-out | HIGH | Open | src/pages/events/utils/event.utils.ts | 218-244 |
| 3 | Videos tab omits controlled column `sortOrder` | MEDIUM | Open | src/pages/events/onboarding/components/event-view/event-view-related-table.tsx | 78-88 |
| 4 | Athletes/crew controlled server `sortOrder` (superseded) | MEDIUM | ✅ Fixed | src/pages/events/onboarding/constants.ts | 134-166 |
| 5 | Add Crew modal duplicates Add Athlete modal | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-view/event-view-add-related-modal.tsx | 1-78 |
| 6 | Filter UI placeholder | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-view/event-view-related-actions.tsx | 1-22 |

### Re-review notes (2026-06-08 @ `fb6d13d1`)

| Change | Verdict |
|--------|---------|
| `432529b2` client-side athletes/crew lists | **New HIGH** — unbounded `fetchAllEventRelatedList` |
| `432529b2` client comparator sorters | Resolves prior athletes/crew `sortOrder` finding |
| `ef48d4e9` / `getEvent` rename | Consistent with backend `getEventByRef` |
| `de4d86a2` remove unused table sort props | Cleanup; superseded by `432529b2` architecture |
| Dashboard titles | **Accepted** |

**Positive notes:** Videos tab uses `useServerTableControls` + `PaginationBar` + `applyServerSort`; athletes/crew use `usePagination` `hidden`/`bar` for client paging; crew CRUD + shared add modal; `DestructiveActionConfirmModal` for removes; `buildEventViewPath` reused in app-user linked events.

**Merge readiness:** **Not merge-ready** — 1 open High + 1 open Medium on frontend. Backend has no open blockers.
