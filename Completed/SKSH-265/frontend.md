# Frontend PR Review — skillshow-admin-ui (`SKSH-265`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-01 @ `4e59b64e`  
**Scope:** Crew dashboard, editor feedback, admin insights tabs, internal revision UI, edit-request detail, realtime/unread (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 5 (0 Critical, 1 High, 1 Medium) — **0 Open**, **5 Fixed**

### Protected modules

No changes in this PR to `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx`, or `AuditLogTable.tsx`.

---

## GitHub comments (Open findings)

_None — all findings resolved on branch._

---

---
Protected module changed: `useServerTableControls`

Risk Level: CRITICAL  
File Path: src/hooks/use-server-table-controls.ts  
Lines: 6-40

Description:
**Protected module.** Prior revision added `searchControls` to the frozen hook. Latest branch reverts that diff; admin list uses `usePagination` + URL-synced search via `useAdminEditRequestListControls` instead.

**Re-verification (@ `4e59b64e`):** `use-server-table-controls.ts` unchanged vs `main`.

**Status:** ✅ Fixed

---

---
Admin list tables duplicate pagination footer wiring

Risk Level: MEDIUM  
File Path: src/pages/adminEditRequest/hooks/use-admin-edit-request-list-pagination.ts  
Lines: 12-38

Description:
**DRY / Contract.** Shared `useAdminEditRequestListPagination` + `AdminEditRequestListPaginationFooter` now supply hidden table pagination and `PaginationBar` props for both `admin-edit-request-table.tsx` and `admin-edit-request-insights-table.tsx`.

**Re-verification (@ `4e59b64e`):** Single feature-level helper; no duplicated footer markup.

**Status:** ✅ Fixed

---

---
Edit-request detail loads each source video with a separate API call

Risk Level: HIGH  
File Path: src/pages/editRequest/hooks/useEditRequestDetail.ts  
Lines: 40-50

Description:
Detail hook loads via `loadEditRequestDetailWithOutputs` and maps `backendRequest.sourceVideos` with `mapEditRequestSourceVideoSummaries` — no per-video `getVideo` fan-out.

**Re-verification (@ `4e59b64e`):** N+1 video fetch removed; backend embeds summaries.

**Status:** ✅ Fixed

---

---
Admin edit-request lists bypass server pagination contract

Risk Level: HIGH  
File Path: src/pages/adminEditRequest/index.tsx  
Lines: 113-117

Description:
**Contract / Global consistency.** Page uses `usePagination(listPaginationFilterKey, 1)` where `listPaginationFilterKey` includes tab, filters, and debounced URL search — page resets on filter/tab/search changes. Allowed alternative when server sort is not used (`usePagination` per contract).

**Re-verification (@ `4e59b64e`):** Hand-rolled pagination removed; protected `use-server-table-controls.ts` not modified.

**Status:** ✅ Fixed

---

---
Crew dashboard and insights tabs stale on socket events

Risk Level: MEDIUM  
File Path: src/utils/edit-request-realtime.utils.ts  
Lines: 278-338

Description:
`refetchActiveEditRequestSurfaces` calls `refetchCrewDashboardSurface` and `refetchAdminEditRequestListSurface` on socket events via global `useEditRequestSocketRealtime`.

**Re-verification (@ `4e59b64e`):** Crew dashboard + insights query keys included in sync path.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed: `useServerTableControls` | CRITICAL | ✅ Fixed | src/hooks/use-server-table-controls.ts | — (reverted) |
| 2 | Admin list tables duplicate pagination footer wiring | MEDIUM | ✅ Fixed | src/pages/adminEditRequest/hooks/use-admin-edit-request-list-pagination.ts | 12-38 |
| 3 | Edit-request detail loads each source video with a separate API call | HIGH | ✅ Fixed | src/pages/editRequest/hooks/useEditRequestDetail.ts | 40-50 |
| 4 | Admin edit-request lists bypass server pagination contract | HIGH | ✅ Fixed | src/pages/adminEditRequest/index.tsx | 113-117 |
| 5 | Crew dashboard and insights tabs stale on socket events | MEDIUM | ✅ Fixed | src/utils/edit-request-realtime.utils.ts | 278-338 |

## Positive notes

- **Global consistency (good):** `refetchActiveEditRequestSurfaces` + `useEditRequestSocketRealtime` unify athlete, admin/editor, crew, insights, and dashboard refresh.
- Crew/editor routing and role gating consistent; editor feedback gates on `myLatestFeedback`.
- **Contract (good):** Output delete uses `DestructiveActionConfirmModal`; export uses `fetchAllPaginatedListItems`; shared pagination helper composes `PaginationBar`.
- Activity history uses `ActivityHistoryTimeline` (domain timeline — `AuditLogTable` N/A).
- URL-synced tab/search via `useAdminEditRequestListControls` improves shareable admin list state without editing protected hooks.

**Merge readiness:** **Merge-ready** — no open Critical/High/Medium blockers. Backend [validatedQuery](./backend.md) finding accepted.
