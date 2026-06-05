# Frontend PR Review — skillshow-admin-ui (`SKSH-271`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-271`  
**Base:** `main...HEAD` (28 files, ~1245 lines)  
**Re-verified:** 2026-06-05 — @ `origin/SKSH-271` → `20ebd747`  
**Scope:** DRY/KISS, global consistency, React performance/hooks, JSX/props, file structure (Critical, High, Medium)  
**Findings:** 10 (0 Critical, 3 High, 7 Medium) — **0 Open**, **2 Fixed**, **8 Accepted**

---

---
`useMemo` dependency arrays include entire `props` object

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-table.tsx  
Lines: 32-95

**Re-verification:**  
Refactored to tab-scoped `athletesTab` / `videosTab` memos; `pagedModel` and `paginationConfig` depend on those slices.

---

---
Event athlete count stale after add (cache sync)

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/events/onboarding/hooks/use-event-view-related-lists.ts  
Lines: 83-101

**Re-verification:**  
`refreshAthletesAfterAdd(added)` awaits invalidate + refetch on athletes queries; modal passes `data.added`. In-tab list and total update correctly.

---

---
`countForTab` shows stale count when API total is zero

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-section.tsx  
Lines: 22-35, 103

**Description:**  
`athletesTableTotal > 0 ? athletesTableTotal : registeredAthleteCount` falls back to static count when API returns `0`.

**Accepted reason:** Deferred per review decision — not blocking SKSH-271 merge.

---

---
`patchEventsListAthletesCount` patches an unused cache key

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/pages/events/onboarding/hooks/use-event-view-related-lists.ts  
Lines: 72-80

**Description:**  
Patches `EVENTS_LIST_ALL_QUERY_KEY`; dashboard uses `EVENTS_LIST_QUERY_KEY`.

**Accepted reason:** Deferred per review decision — events list/detail count sync is follow-up.

---

---
Manual debounce duplicates `useDebouncedValue`

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: `event-view-related-section.tsx`, `event-view-add-athlete-modal.tsx`  
Lines: 15-18, 52-55

**Accepted reason:** Deferred per review decision.

---

---
Add/remove mutations lack user-visible error handling

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: `event-view-add-athlete-modal.tsx`, `use-event-view-related-lists.ts`  
Lines: 52-61, 104-110

**Accepted reason:** Deferred per review decision.

---

---
`video-list.utils.ts` and `VIDEO_UPLOAD_SOURCE` unused in PR

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: `video-list.utils.ts`, `video-list.constants.ts`  
Lines: 1-16, 27-33

**Accepted reason:** Deferred per review decision.

---

---
Remove-athlete confirm copy missing punctuation

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: src/pages/events/onboarding/constants.ts  
Lines: 55-57

**Accepted reason:** Deferred per review decision.

---

---
Add Athlete modal regressed from `ResponsiveModal` to plain `Modal`

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: `event-view-add-athlete-modal.tsx`  
Lines: 64-79

**Accepted reason:** Deferred per review decision.

---

---
`invalidateQueries` + `refetchQueries` is redundant

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: `use-event-view-related-lists.ts`  
Lines: 83-87

**Accepted reason:** Deferred per review decision.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `useMemo` depends on entire `props` object | HIGH | ✅ Fixed | `event-view-related-table.tsx` | 32-95 |
| 2 | Athlete count stale after add | MEDIUM | ✅ Fixed | `use-event-view-related-lists.ts` | 83-101 |
| 3 | `countForTab` stale when API total is 0 | HIGH | Accepted | `event-view-related-section.tsx` | 22-35, 103 |
| 4 | `patchEventsListAthletesCount` patches unused cache key | HIGH | Accepted | `use-event-view-related-lists.ts` | 72-80 |
| 5 | Manual debounce vs `useDebouncedValue` | MEDIUM | Accepted | `event-view-related-section.tsx`, `event-view-add-athlete-modal.tsx` | 15-18, 52-55 |
| 6 | Mutations lack error toasts | MEDIUM | Accepted | `event-view-add-athlete-modal.tsx`, `use-event-view-related-lists.ts` | 52-61, 104-110 |
| 7 | Unused `video-list.utils` / upload-source constants | MEDIUM | Accepted | `video-list.utils.ts`, `video-list.constants.ts` | 1-16, 27-33 |
| 8 | Remove confirm copy punctuation | MEDIUM | Accepted | `onboarding/constants.ts` | 55-57 |
| 9 | `ResponsiveModal` → `Modal` regression | MEDIUM | Accepted | `event-view-add-athlete-modal.tsx` | 64-79 |
| 10 | Redundant invalidate + refetch | MEDIUM | Accepted | `use-event-view-related-lists.ts` | 83-87 |

**Positive notes:** Clean API layer (`eventService.ts`, `event.utils.ts`). Shared columns. Lazy queries gated by `expanded` + `activeTab`. Session mock removed.

**Skipped (per request):** Pagination-related issues; full-review findings #3–10 accepted for this ticket.

**Merge readiness:** ✅ Ready — no open findings (accepted items deferred).
