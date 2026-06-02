# Frontend PR Review — skillshow-admin-ui (`SKSH-116`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-116-main`  
**Base:** `main...HEAD`  
**Re-verified:** `c223f39f` (`fix: bugs`)  
**Scope:** Code introduced or materially changed on this branch (Critical, High, Medium)  
**Findings:** 12 in scope (0 Critical, 3 High, 9 Medium) — **11 fixed**, **1 accepted**, **0 open**, **0 new Critical/High/Medium** (#1 list assign, #7 `columnHandlers` memo out of scope)

---

---
Source save does not refresh detail cache

Risk Level: MEDIUM  
File Path: src/pages/editRequest/edit-details.tsx  
Lines: 92, 109

Description:
After saving source removals/replacements, the participant detail query was not invalidated, so the UI could show stale server state.

Impact:
- User sees outdated videos/reviews after a successful save

Recommendation:
Call `await invalidateDetail()` after successful save and in the error path.

**PR comment (line 92):** **Medium:** Invalidate edit-request detail after source save (success and failure) so the page reflects server state.

**Re-review:** ✅ Fixed @ `6395ced1` — `invalidateDetail()` on success and in `catch`.

---

---
Source save batch not atomic

**Status: Accepted** — team decision; sequential per-step API with error UX is sufficient for SKSH-116.

Risk Level: MEDIUM  
File Path: src/pages/editRequest/edit-details.tsx  
Lines: 59-110

**Also:** src/pages/editRequest/utils/edit-request-source-files.utils.ts  
**Lines:** 15-29

Description:
Participant “Update” runs one `mutateAsync` per removal and replacement. A mid-batch server failure can leave partial persisted state. `6395ced1` added `setSourceSaveError` + `formatEditRequestSourceSaveError`, tracks completed steps, and trims local pending removals/replacements for steps that already succeeded (~96–108).

Impact:
- Partial server state if a later step fails (no server transaction)
- Mitigated by visible errors and local queue sync with completed steps

Recommendation:
Optional follow-up: single batch backend action. Not required for SKSH-116.

**Re-review:** Accepted @ `6395ced1` — not pursuing atomic batch for this PR.

---

---
Create output leaves orphan record if upload fails

Risk Level: MEDIUM  
File Path: src/pages/adminEditRequest/hooks/use-admin-edit-request-output-actions.ts  
Lines: 96-105

Description:
`createOutputWithVersionsMutation` created the output before upload; upload failure left an empty output row.

Impact:
- Empty output records until manual cleanup

Recommendation:
Delete the output in the upload `catch` before rethrowing.

**PR comment (line 100):** **Medium:** On upload failure after create, call `deleteAdminEditRequestOutput` before surfacing the error.

**Re-review:** ✅ Fixed @ `6395ced1` — `deleteAdminEditRequestOutput` in catch (delete errors swallowed with `.catch(() => undefined)`; rare double-failure edge only).

---

---
List socket toast + global refetch

Risk Level: HIGH  
File Path: src/pages/adminEditRequest/hooks/use-admin-edit-request-socket.ts  
Lines: 28-34

**Also:** src/utils/edit-request-realtime.utils.ts  
**Lines:** 26-47

Description:
Admin socket handler previously called `refetchAllEditRequestQueries`, refetching participant queries and every active admin detail on the list route.

Impact:
- Extra API traffic and list/detail flicker

Recommendation:
List route: `refetchAdminEditRequestListQueries` only. Detail route: `refetchAdminEditRequestQueries` for that id.

**PR comment (line 33):** **High:** Narrow list-route socket refresh to admin list query key only.

**Re-review:** ✅ Fixed @ `6395ced1` — `refresh` branches on `editRequestId`; `refetchAdminEditRequestQueries` no longer refetches all details when id omitted.

---

---
Bulk download no unmount cleanup

Risk Level: HIGH  
File Path: src/pages/adminEditRequest/hooks/use-admin-edit-request-source-downloads.ts  
Lines: 64-67

Description:
Bulk download did not abort in-flight work or close progress UI on unmount.

Impact:
- Memory leaks and setState-after-unmount on navigation away

Recommendation:
On unmount, call `closeDownloadAllProgress` and `abortAll`.

**Re-review:** ✅ Fixed — unchanged and still verified @ `6395ced1`.

---

---
Source files sync key misses video-only updates

Risk Level: HIGH  
File Path: src/pages/editRequest/components/EditDetailsSourceFilesSection.tsx  
Lines: 148-154

**Also:** src/pages/editRequest/utils/edit-request-source-files.utils.ts  
**Lines:** 32-46

Description:
`sourceSyncKey` was only `id|status`, so video/review-only updates did not reset local rows.

Impact:
- Stale local source rows after admin review updates

Recommendation:
Fingerprint videos and `videoReviews`; reset rows in `useEffect` when the key changes.

**PR comment (line 148):** **High:** Extend source sync key with video/review fingerprint and reset in `useEffect`.

**Re-review:** ✅ Fixed @ `6395ced1` — `buildEditRequestSourceSyncKey` + `useEffect([sourceSyncKey])`.

---

---
Admin media preview without abort / mounted guard

Risk Level: HIGH  
File Path: src/pages/adminEditRequest/hooks/use-admin-edit-request-media-view.ts  
Lines: 1

**Also:** src/hooks/use-edit-request-media-view.ts  
**Lines:** 14-74

Description:
Async preview URL fetch could complete after close/unmount and call `setState`.

Impact:
- React warnings and stale preview URLs

Recommendation:
Generation ref; invalidate on unmount and before a new open.

**Re-review:** ✅ Fixed @ `6395ced1` — shared `useEditRequestMediaView`; admin hook re-exports it.

---

---
Participant output preview without abort / mounted guard

Risk Level: HIGH  
File Path: src/pages/editRequest/components/EditDetailsOutputSection.tsx  
Lines: 316-340, 437-444

Description:
`handleViewFullscreen` fetched a view URL without a generation or mounted guard when the modal closed mid-load.

Impact:
- setState-after-unmount or stale URL on slow networks

Recommendation:
Reuse generation-ref preview hook.

**PR comment (line 337):** **High:** Use shared media view hook with generation guard for participant fullscreen preview.

**Re-review:** ✅ Fixed @ `6395ced1` — `useEditRequestMediaView` + `VideoFullscreenModal` wired to `mediaView`.

---

---
Monolithic source-files section (admin)

Risk Level: MEDIUM  
File Path: src/pages/adminEditRequest/components/admin-edit-request-source-files-section.tsx  
Lines: (split across hook + row)

Description:
Admin source-files UI was a single large component mixing download logic and row rendering.

Impact:
- Harder to test and reuse; unnecessary re-renders

Recommendation:
Split into hook + row component.

**Re-review:** ✅ Fixed.

---

---
Participant realtime hits admin queries

Risk Level: MEDIUM  
File Path: src/hooks/use-edit-request-realtime.ts  
Lines: 33, 64

Description:
Participant socket handler refetched admin edit-request query keys.

Impact:
- Wasted admin API calls for athlete sessions

Recommendation:
Use `refetchParticipantEditRequestQueries` only.

**Re-review:** ✅ Fixed.

---

---
Duplicated output-actions hook instances

Risk Level: MEDIUM  
File Path: src/pages/adminEditRequest/components/admin-edit-request-output-management-section.tsx  
Lines: 35-40

Description:
`useAdminEditRequestOutputActions` was mounted per output item.

Impact:
- Redundant hook instances and confusing loading state

Recommendation:
Lift hook once in the management panel.

**Re-review:** ✅ Fixed.

---

---
setState during render in source files section

Risk Level: MEDIUM  
File Path: src/pages/editRequest/components/EditDetailsSourceFilesSection.tsx  
Lines: 151-154

Description:
Row reset ran `setState` in the render body when `sourceSyncKey` changed.

Impact:
- Strict-mode warnings; unpredictable updates under concurrent React

Recommendation:
Move reset into `useEffect` keyed on `sourceSyncKey`.

**Re-review:** ✅ Fixed @ `6395ced1` — `useEffect` on `sourceSyncKey` (~151–154).

---

---
setState during render on route id change

Risk Level: MEDIUM  
File Path: src/pages/editRequest/edit-details.tsx  
Lines: 41-45

Description:
Route `id` change reset local source state during render.

Impact:
- Strict-mode warnings

Recommendation:
`useEffect(() => { … }, [id])`.

**Re-review:** ✅ Fixed @ `6395ced1` — `useEffect` clears replacements/removals/error on `[id]`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 2 | Source save does not refresh detail cache | MEDIUM | ✅ Fixed | `edit-details.tsx` | 92, 109 |
| 3 | Source save batch not atomic | MEDIUM | Accepted | `edit-details.tsx` | 59-110 |
| 4 | Orphan output if upload fails | MEDIUM | ✅ Fixed | `use-admin-edit-request-output-actions.ts` | 96-105 |
| 5 | List socket toast + global refetch | HIGH | ✅ Fixed | `use-admin-edit-request-socket.ts` | 28-34 |
| 6 | Bulk download no unmount cleanup | HIGH | ✅ Fixed | `use-admin-edit-request-source-downloads.ts` | 64-67 |
| 8 | Source sync key too narrow | HIGH | ✅ Fixed | `EditDetailsSourceFilesSection.tsx` | 148-154 |
| 9a | Admin media preview no abort guard | HIGH | ✅ Fixed | `use-edit-request-media-view.ts` | 14-74 |
| 9b | Participant output preview no abort guard | HIGH | ✅ Fixed | `EditDetailsOutputSection.tsx` | 316-444 |
| 10 | Monolithic source-files section (admin) | MEDIUM | ✅ Fixed | admin source-files split | — |
| 11 | Participant realtime hits admin queries | MEDIUM | ✅ Fixed | `use-edit-request-realtime.ts` | — |
| 12 | Duplicated output-actions hook | MEDIUM | ✅ Fixed | `admin-edit-request-output-management-section.tsx` | 35-40 |
| 13 | setState during render (source section) | MEDIUM | ✅ Fixed | `EditDetailsSourceFilesSection.tsx` | 151-154 |
| 14 | setState during render (edit-details id) | MEDIUM | ✅ Fixed | `edit-details.tsx` | 41-45 |

**Positive notes:** Re-reviewed @ `c223f39f`. **11 fixed**, **1 accepted** (#3), **0 open**. Prior fixes verified unchanged in edit-request core paths (source save, socket refetch, output rollback, sync key, media preview guard). Latest commit adjusts `VideoFullscreenModal` layout and activity-history copy only — `onClose` still wired to `mediaView.closeView`; no regression to #9b. Merged main/SKSH-295/sksh-258 changes outside edit-request review scope unless they touch listed files. **0 new** Critical/High/Medium in SKSH-116 edit-request scope.

**Skipped:** Ad-hoc palette / Ant `!` overrides. Pagination. #1 list assign, #7 `columnHandlers` memo.
