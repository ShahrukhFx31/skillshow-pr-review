# Frontend PR Review — skillshow-admin-ui (`SKSH-271`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-271`  
**Base:** `main...HEAD`  
**Re-reviewed:** 2026-06-01 (after developer fixes)  
**Scope:** React performance, hooks, memory, architecture, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 2 (0 Critical, 1 High, 1 Medium) — **2 Open**, **0 Fixed**

---

---
`useMemo` dependency arrays include entire `props` object

Risk Level: HIGH  
**Status:** Open  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-table.tsx  
Lines: 48-94

**Description:**  
`pagedModel` and `paginationConfig` still list the whole `props` object in dependency arrays (`[..., props, tab]`). Parent passes a new props object each render, so memos recompute every time.

**Re-verification:**  
Unchanged on latest `origin/SKSH-271` (`de56991d`). No commit touches this file after the initial event-view refactor.

**Recommendation:**  
Depend on specific `props.*` fields used inside each `useMemo`.

**PR comment (line 62):**  
**High:** `useMemo` still depends on the full `props` object. Please list specific prop fields so memoization stays effective.

---

---
Event athlete count may stay stale after add

Risk Level: MEDIUM  
**Status:** Open  
File Path: src/pages/events/onboarding/hooks/use-event-view-related-lists.ts  
Lines: 110-113

**Description:**  
`refreshAthletesAfterAdd` still awaits `invalidateQueries`, then calls `syncEventsListAthletesCountFromCache()` immediately. Invalidation does not wait for refetch completion, so sync may read stale `pagination.total`.

**Re-verification:**  
Unchanged on latest `origin/SKSH-271`. `EventViewAddAthleteModal` receives `data.added` from the API but only calls `onSuccess()` without passing the count — optimistic patch with `+data.added` is still unused.

**Recommendation:**  
`await queryClient.refetchQueries({ queryKey: eventViewQueryKeys.athletesAll(eventRouteId) })` then sync, or `patchEventsListAthletesCount(data.added)` in `refreshAthletesAfterAdd` (wire `added` from the modal).

**PR comment (line 110):**  
**Medium:** Add-athlete refresh still syncs count from cache right after invalidate. Consider `refetchQueries` + await or optimistic `+added` from the add response.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `useMemo` depends on entire `props` object | HIGH | Open | `event-view-related-table.tsx` | 48-94 |
| 2 | Athlete count stale after add (cache sync race) | MEDIUM | Open | `use-event-view-related-lists.ts` | 110-113 |

**Positive notes:** Data/actions in `useEventViewRelatedLists`. Columns in `event-view-related-columns.tsx`. Scoped athlete invalidation (not full catalog refetch). Remove path uses optimistic `patchEventsListAthletesCount(-1)`.

**New issues on re-review:** None (Critical/High/Medium beyond the two above).

**Skipped (per request):** Pagination-related issues in list endpoints.
