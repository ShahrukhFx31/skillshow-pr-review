# Frontend PR Review — skillshow-admin-ui (`SKSH-326`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-326`  
**Base:** `main...HEAD` @ `39af45cf`  
**Re-verified:** 2026-06-09  
**Scope:** Unify upload/edit event dropdown on `GET /v1/events/me` for athletes and coaches; remove coach-only video-library lookups path (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 3 (0 Critical, 0 High, 3 Medium) — **2 Fixed**, **1 Accepted**

### Protected modules

No changes to `use-server-table-controls.ts`, `pagination-bar.tsx`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx`, or `AuditLogTable.tsx`.

---

## Re-verification (`39af45cf`)

| # | Finding | Status | Evidence |
|---|---------|--------|----------|
| 1 | Unrelated video-library Delete menu styling (scope) | **Accepted** | Still bundled in PR; updated to `danger: true` + `mdi:delete-outline` size 16 — matches `event-columns.tsx`, `partners-columns.tsx`, and management table menus |
| 2 | `buildUploadEventSelectOptions` pass-through wrapper (KISS) | **✅ Fixed** | Wrapper removed; hook calls `mapAthleteRegisteredEventsToSelectOptions` directly |
| 3 | Event-picker comments imply role-filtered API (Contract) | **✅ Fixed** | JSDoc aligned to “all active platform events for the upload event picker” across `eventService.ts`, `constants.ts`, `types.ts`, hook, and `video.types.ts` |

---

## GitHub comments (resolved)

### 1. `src/pages/videoLibrary/dashboard/components/video-library-columns.tsx` line 47

**Accepted (scope):** Delete menu change remains outside event-picker scope, but now follows the established `danger: true` + `mdi:delete-outline` pattern used elsewhere — acceptable bundled cleanup.

### 2. `src/pages/events/utils/event.utils.ts` — removed

**✅ Fixed:** `buildUploadEventSelectOptions` and `mapVideoLibraryEventsToSelectOptions` deleted; hook inlines `mapAthleteRegisteredEventsToSelectOptions`.

### 3. `src/pages/events/types.ts` line 189

**✅ Fixed:** Comments now match backend swagger — active platform events for the upload picker, not role-filtered lists.

---

---
Unrelated video-library Delete menu styling (scope)

Risk Level: MEDIUM  
File Path: src/pages/videoLibrary/dashboard/components/video-library-columns.tsx  
Lines: 47-50

Description:
This ticket refactors upload/edit event selection under `events/**` and `videos/**`, but also changes the video-library action dropdown Delete item (from manual red icon/label styling to Ant Design `danger: true` with `mdi:delete-outline`).

Impact:
- Extra surface area for QA outside event-picker scope.
- Minor rollback coupling if event-picker work is held.

Recommendation:
Ideally split to a separate PR; acceptable when aligned with codebase delete-menu conventions.

**Re-verification (`39af45cf`):** Change now matches `event-columns.tsx`, `partners-columns.tsx`, and management dashboard column menus (`danger: true`, `mdi:delete-outline`, size 16). Bundled scope creep, but consistent with **Global consistency** patterns.

**Status:** Accepted — intentional delete-menu alignment with established table action dropdowns.

---

---
`buildUploadEventSelectOptions` is a pass-through wrapper (KISS)

Risk Level: MEDIUM  
File Path: src/pages/events/utils/event.utils.ts  
Lines: (removed)  
File Path: src/pages/videos/hooks/useUploadEventSelectOptions.ts  
Lines: 6, 24-27

Description:
**KISS:** After removing the coach/athlete branch, `buildUploadEventSelectOptions` only delegated to `mapAthleteRegisteredEventsToSelectOptions`.

**Re-verification (`39af45cf`):** Wrapper and `BuildUploadEventSelectOptionsParams` removed. Hook imports and calls `mapAthleteRegisteredEventsToSelectOptions(myEvents)` directly.

**Status:** ✅ Fixed

---

---
Event-picker comments imply role-filtered API; backend returns all active events (Contract)

Risk Level: MEDIUM  
File Path: src/pages/events/types.ts  
Lines: 189  
File Path: src/api/services/eventService.ts  
Lines: 49  
File Path: src/pages/videos/hooks/useUploadEventSelectOptions.ts  
Lines: 10

Description:
**Contract:** Comments previously described role-specific event lists; `GET /v1/events/me` returns all active platform events for any authenticated uploader.

**Re-verification (`39af45cf`):** JSDoc updated consistently:
- `eventService.ts`: “All active platform events for the upload event picker”
- `constants.ts`: “all active platform events for the upload event picker”
- `types.ts`: “active platform events for the upload event picker”
- `useUploadEventSelectOptions.ts`: “all active platform events from `GET /v1/events/me`”
- `video.types.ts`: “Active platform events for the Event field (`GET /v1/events/me`)”
- `eventsQueryKeys.myRegisteredEvents` inline comment added

**Status:** ✅ Fixed

---

## Positive notes

- **DRY:** Single API source (`listMyRegisteredEvents`) and mapper for athletes and coaches; removed parallel `getVideoLibraryLookups` fetch.
- **KISS:** Hook simplified to one `useQuery`, direct mapper call, smaller `event.utils.ts`.
- **Contract:** JSDoc now matches backend `listEventsForViewer` → `findAllActiveEvents` behavior.
- **Global consistency:** Video-library delete menu aligned with other table action dropdowns.
- **Protected modules:** No edits to frozen infrastructure.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unrelated video-library Delete menu styling (scope) | MEDIUM | Accepted | src/pages/videoLibrary/dashboard/components/video-library-columns.tsx | 47-50 |
| 2 | `buildUploadEventSelectOptions` is a pass-through wrapper (KISS) | MEDIUM | ✅ Fixed | src/pages/videos/hooks/useUploadEventSelectOptions.ts | 6, 24-27 |
| 3 | Event-picker comments imply role-filtered API; backend returns all active events (Contract) | MEDIUM | ✅ Fixed | src/pages/events/types.ts | 189 |

**Merge readiness:** No open Critical/High/Medium blockers. All findings resolved (2 Fixed, 1 Accepted). **Merge-ready.**
