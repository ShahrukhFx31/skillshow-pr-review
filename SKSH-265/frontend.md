# Frontend PR Review — skillshow-admin-ui (`SKSH-265`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-08 @ `bf68d76b`  
**Scope:** Crew dashboard, editor feedback, admin insights tabs, internal revision UI, edit-request detail, realtime/unread (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 5 (0 Critical, 2 High, 1 Medium, 2 Fixed)

> **Scope note:** Pagination / list-query performance findings are omitted per review request.

### Protected modules

No changes in this PR to `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx`, or `AuditLogTable.tsx`.

---

## GitHub comments (Open findings)

### 1. `src/pages/editRequest/hooks/useEditRequestDetail.ts` line 58

**High:** Detail hook still fans out one `getVideo` per source file (up to 20) — can we batch-fetch or embed video summaries on the edit-request detail response to avoid N parallel calls per page view?

### 2. `src/pages/adminEditRequest/index.tsx` line 93

**High:** Tabbed server lists still hand-roll `page`/`pageSize` — please adopt `useServerTableControls` + `PaginationBar` (as in `partners/dashboard`) so tab/filter changes reset page via `filterKey` and match the protected list contract.

### 3. `src/pages/adminEditRequest/components/admin-edit-request-insights-table.tsx` line 49

**Medium:** Insights table duplicates rows-per-page + `Pagination` markup — please compose `PaginationBar` / `usePagination.bar` instead of copying the footer from `admin-edit-request-table.tsx`.

---

---
Edit-request detail loads each source video with a separate API call

Risk Level: HIGH  
File Path: src/pages/editRequest/hooks/useEditRequestDetail.ts  
Lines: 54-83

Description:
`useEditRequestDetail` runs `Promise.allSettled(requestedVideoIds.map((vid) => getVideo(vid)))`. Backend allows up to 20 videos per request, so one detail view can trigger up to 20 parallel `GET /videos/:id` calls on every load/refetch. `allSettled` avoids total failure but not round-trip count.

Impact:
- Slow detail load and extra server load for multi-video requests
- Partial failures drop videos from the resolved list silently

Recommendation:
Batch videos-by-ids API or embed trimmed video metadata on `GET /edit-requests/:id`.

**PR comment (line 58):**  
**High:** Detail hook still fans out one `getVideo` per source file (up to 20) — can we batch-fetch or embed video summaries on the edit-request detail response?

**Status:** Open

---

---
Admin edit-request lists bypass `useServerTableControls` contract

Risk Level: HIGH  
File Path: src/pages/adminEditRequest/index.tsx  
Lines: 93-170

Description:
**Contract / Global consistency.** This PR adds insights and internal-revision tabs, export-all, and unread badges, but list state remains hand-rolled (`useState` for `page`/`pageSize`, manual `setPage(1)` in filter handlers). Server-driven dashboards elsewhere (`partners/dashboard`, `management/app-users/dashboard`, `crew-users/dashboard`) use `useServerTableControls` with `extraFilterState` / `filterKey` for consistent page reset.

Impact:
- Tab/filter changes depend on ad-hoc `useEffect` resets; new tabs can miss a reset path
- Pagination UI duplicated in child tables instead of `PaginationBar` + `usePagination.hidden`

Recommendation:
Refactor `index.tsx` to `useServerTableControls` with `extraFilterState: { viewMode, paymentStatusFilter, requestedByFilter, createdDateRange }`; pass `bar`/`hidden` to `AdminEditRequestTable` and `AdminEditRequestInsightsTable`.

**PR comment (line 93):**  
**High:** Tabbed server lists still hand-roll `page`/`pageSize` — please adopt `useServerTableControls` + `PaginationBar` so tab/filter changes reset page via `filterKey`.

**Status:** Open

---

---
Insights table duplicates pagination footer markup

Risk Level: MEDIUM  
File Path: src/pages/adminEditRequest/components/admin-edit-request-insights-table.tsx  
Lines: 49-74

Description:
**DRY / Contract.** New insights table copies the manual `Select` + `Pagination` footer pattern from `admin-edit-request-table.tsx` instead of composing frozen `PaginationBar` via `usePagination`.

Impact:
- Two copies of rows-per-page UI to maintain when bounds or copy change
- Drifts from protected pagination contract over time

Recommendation:
Accept `bar` props from parent `usePagination` hook or render `<PaginationBar {...bar} />` below the table.

**PR comment (line 49):**  
**Medium:** Please use `PaginationBar` / `usePagination.bar` here instead of duplicating the footer markup from the main admin table.

**Status:** Open

---

---
Crew dashboard does not refresh on edit-request realtime events

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/utils/edit-request-realtime.utils.ts  
Lines: 288-303

**Re-verification (@ `bf68d76b`):** `refetchActiveEditRequestSurfaces` refetches `crewDashboardQueryKey` (line 302). Global `useEditRequestSocketRealtime` (via `admin-edit-request-unread-summary-bridge.tsx` in dashboard layout) calls `syncEditRequestRealtimeQueries` on socket events.

**PR comment:** Resolved on branch.

---

---
Admin insights tabs omit socket-driven cache invalidation

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/utils/edit-request-realtime.utils.ts  
Lines: 254-267

**Re-verification (@ `bf68d76b`):** `refetchAdminEditRequestListSurface` refetches insights assignment-returns and feedbacks query keys; `invalidateCachedAdminEditRequestListQueries` marks them stale.

**PR comment:** Resolved on branch.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Edit-request detail loads each source video with a separate API call | HIGH | Open | src/pages/editRequest/hooks/useEditRequestDetail.ts | 54-83 |
| 2 | Admin edit-request lists bypass `useServerTableControls` contract | HIGH | Open | src/pages/adminEditRequest/index.tsx | 93-170 |
| 3 | Insights table duplicates pagination footer markup | MEDIUM | Open | src/pages/adminEditRequest/components/admin-edit-request-insights-table.tsx | 49-74 |
| 4 | Crew dashboard does not refresh on edit-request realtime events | MEDIUM | ✅ Fixed | src/utils/edit-request-realtime.utils.ts | 288-303 |
| 5 | Admin insights tabs omit socket-driven cache invalidation | MEDIUM | ✅ Fixed | src/utils/edit-request-realtime.utils.ts | 254-267 |

## Positive notes

- **Global consistency (good):** `refetchActiveEditRequestSurfaces` + `useEditRequestSocketRealtime` unify athlete, admin/editor, crew, insights, and dashboard refresh; deprecated per-page socket hooks consolidated.
- Crew/editor routing and role gating consistent; editor feedback gates on `myLatestFeedback`.
- **Contract (good):** Output delete uses `DestructiveActionConfirmModal`; export uses shared `fetchAllPaginatedListItems`.
- Activity history uses `ActivityHistoryTimeline` (domain timeline, not audit-log list — `AuditLogTable` N/A).
- **Protected modules:** None modified in this PR.

**Merge readiness:** **Not merge-ready** — 2 open High + 1 open Medium on frontend; 4 open High on backend. Realtime findings fixed on branch.
