# Frontend PR Review — skillshow-admin-ui (`SKSH-265`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Scope:** Crew dashboard, editor feedback, admin insights tabs, internal revision UI, edit-request detail, realtime/unread (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract enforced)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 4 (0 Critical, 1 High, 1 Medium) — **2 Open**, **2 Fixed**

> **Scope note:** Pagination / list-query performance findings are omitted per review request.

---

## GitHub comments (Open findings)

### 1. `src/pages/editRequest/hooks/useEditRequestDetail.ts` line 58

**High:** Detail hook still fans out one `getVideo` per source file (up to 20) — can we batch-fetch or embed video summaries on the edit-request detail response to avoid N parallel calls per page view?

### 2. `src/pages/adminEditRequest/index.tsx` line 93

**Medium:** This PR adds insights/internal-revision tab lists but keeps hand-rolled `page`/`pageSize` state — please adopt `useServerTableControls` + `PaginationBar` (as in `partners/dashboard`) so tab/filter changes reset page via `filterKey` and pagination matches the protected contract.

---

---
Edit-request detail loads each source video with a separate API call

Risk Level: HIGH  
File Path: src/pages/editRequest/hooks/useEditRequestDetail.ts  
Lines: 55-83

Description:
`useEditRequestDetail` resolves `requestedVideoIds` then runs `Promise.allSettled(requestedVideoIds.map((vid) => getVideo(vid)))`. The backend allows up to 20 videos per request, so a single detail view can trigger up to 20 parallel `GET /videos/:id` calls on every load and refetch. `allSettled` avoids one failure aborting the batch but does not reduce round-trips.

Impact:
- Slow detail page load and extra server load for multi-video requests
- Partial failures still drop videos silently from the resolved list

Recommendation:
Add a batch videos-by-ids API (or include trimmed video metadata on `GET /edit-requests/:id`) and fetch once.

**PR comment (line 58):**  
**High:** Detail hook still fans out one `getVideo` per source file (up to 20) — can we batch-fetch or embed video summaries on the edit-request detail response to avoid N parallel calls per page view?

**Status:** Open

---

---
Admin edit-request lists bypass `useServerTableControls` contract

Risk Level: MEDIUM  
File Path: src/pages/adminEditRequest/index.tsx  
Lines: 93-103, 156-166

Description:
**Contract / Global consistency.** This PR expands My Edit Requests with insights and internal-revision tabs and export-all, but list state remains hand-rolled (`useState` for `page`/`pageSize`, manual `setPage(1)` in several filter handlers). Other server-driven dashboards in the same codebase (`partners/dashboard`, `management/app-users/dashboard`, etc.) use `useServerTableControls` with `filterKey` to reset page when search/filters/tabs change.

Impact:
- Tab or filter changes rely on ad-hoc `useEffect` resets; easy to miss a dependency when adding tabs (insights were added without the shared hook)
- Pagination footer duplicated in `admin-edit-request-table.tsx` and `admin-edit-request-insights-table.tsx` instead of `PaginationBar` + `usePagination.hidden`

Recommendation:
Refactor `index.tsx` to `useServerTableControls` with `extraFilterState: { viewMode, paymentStatusFilter, ... }` so tab switches reset page consistently; wire tables through `PaginationBar`.

**PR comment (line 93):**  
**Medium:** Please adopt `useServerTableControls` + `PaginationBar` for the expanded tabbed lists so pagination/filter resets match the protected server-list contract.

**Status:** Open

---

---
Crew dashboard does not refresh on edit-request realtime events

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/utils/edit-request-realtime.utils.ts  
Lines: 288-303

Description (original):
Crew home KPI/recent list was not invalidated on edit-request socket events.

**Re-verification:**
`refetchActiveEditRequestSurfaces` now refetches `crewDashboardQueryKey` (line 302). Global socket wiring via `useEditRequestSocketRealtime` in `admin-edit-request-unread-summary-bridge.tsx` (dashboard layout) and edit-request pages calls `syncEditRequestRealtimeQueries` on `EDIT_REQUEST_UPDATED` / reconnect.

**PR comment:** Resolved on branch.

---

---
Admin insights tabs omit socket-driven cache invalidation

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/utils/edit-request-realtime.utils.ts  
Lines: 254-267

Description (original):
Insights assignment-returns / feedbacks query keys were not included in socket-driven refetch.

**Re-verification:**
`refetchAdminEditRequestListSurface` refetches `ADMIN_EDIT_REQUEST_INSIGHTS_LIST_QUERY_KEY.assignmentReturns` and `.feedbacks` (lines 259-266). `invalidateCachedAdminEditRequestListQueries` covers the same keys for stale marking.

**PR comment:** Resolved on branch.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Edit-request detail loads each source video with a separate API call | HIGH | Open | src/pages/editRequest/hooks/useEditRequestDetail.ts | 55-83 |
| 2 | Admin edit-request lists bypass `useServerTableControls` contract | MEDIUM | Open | src/pages/adminEditRequest/index.tsx | 93-103, 156-166 |
| 3 | Crew dashboard does not refresh on edit-request realtime events | MEDIUM | ✅ Fixed | src/utils/edit-request-realtime.utils.ts | 288-303 |
| 4 | Admin insights tabs omit socket-driven cache invalidation | MEDIUM | ✅ Fixed | src/utils/edit-request-realtime.utils.ts | 254-267 |

## Positive notes

- **Global consistency (good):** `refetchActiveEditRequestSurfaces` + `useEditRequestSocketRealtime` unify athlete, admin/editor, crew, insights, and dashboard refresh paths; deprecated per-page socket hooks consolidated.
- Crew/editor dashboard routing and role gating are consistent; editor feedback modal gates on `myLatestFeedback`.
- Insights tables reuse column hooks; internal-revision and crew-return flows colocate constants and align with backend enums.
- **Protected modules:** No changes to `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, or `antd.adapter.tsx`.
- **Contract (good):** Destructive output delete uses `DestructiveActionConfirmModal`; export uses shared `fetchAllPaginatedListItems` util.

**Merge readiness:** **Not merge-ready** — 1 open High (per-video detail fetches) + 1 open Medium (list controls contract). Realtime/insights refresh fixed on branch. Pair with backend open High items before full ticket sign-off.
