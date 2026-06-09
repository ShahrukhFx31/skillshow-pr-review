# Frontend PR Review — skillshow-admin-ui (`SKSH-265`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-01 @ `3ae937f6`  
**Scope:** Crew dashboard, editor feedback, admin insights tabs, internal revision UI, edit-request detail, realtime/unread (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 5 (1 Critical, 1 High, 1 Medium) — **2 Open**, **3 Fixed**

### Protected modules

**Modified in this PR:** `src/hooks/use-server-table-controls.ts` (optional `searchControls` for URL-synced search).

No changes to `pagination-bar.tsx`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx`, or `AuditLogTable.tsx`.

---

## GitHub comments (Open findings)

### 1. `src/hooks/use-server-table-controls.ts` line 17

**Critical:** This PR modifies the frozen `useServerTableControls` hook — please revert and move the `searchControls` extension to a dedicated shared-module ticket, or get explicit approval to change the protected hook before merge.

### 2. `src/pages/adminEditRequest/components/admin-edit-request-table.tsx` line 67

**Medium:** Both admin list tables hand-build hidden `Table` pagination + `PaginationBar` props — please extract a feature-level helper (or expose `bar`/`hidden` from the page’s pagination hook) so `admin-edit-request-table.tsx` and `admin-edit-request-insights-table.tsx` share one footer contract.

---

---
Protected module changed: `useServerTableControls`

Risk Level: CRITICAL  
File Path: src/hooks/use-server-table-controls.ts  
Lines: 6-40

Description:
**Protected module.** This PR adds optional `searchControls` to bypass internal search state so `adminEditRequest/index.tsx` can URL-sync EDR search via `useAdminEditRequestListControls`. The hook is listed as frozen in the review contract; changes require a dedicated ticket or explicit authorization.

Impact:
- Shared list-control behavior changes for every consumer of `useServerTableControls` across the admin UI
- Future pagination/search refactors must account for dual search paths in the protected hook

Recommendation:
Revert the protected-file diff and either (a) open a scoped ticket to extend `useServerTableControls` with `searchControls`, or (b) keep URL search outside the hook by passing debounced search through `extraFilterState` only (no protected-module edit).

**PR comment (line 17):**  
**Critical:** This PR modifies the frozen `useServerTableControls` hook — please revert and move the `searchControls` extension to a dedicated shared-module ticket, or get explicit approval before merge.

**Status:** Open

---

---
Admin list tables duplicate pagination footer wiring

Risk Level: MEDIUM  
File Path: src/pages/adminEditRequest/components/admin-edit-request-table.tsx  
Lines: 67-86

Description:
**DRY / Contract.** `AdminEditRequestTable` and `AdminEditRequestInsightsTable` both manually construct `hiddenPagination` (`style: { display: "none" }`) and a `bar` object for `PaginationBar`. The page already uses `useServerTableControls` (which wraps `usePagination` internally) but does not expose `bar`/`hidden`, so both child tables copy the same ~20-line footer pattern.

Impact:
- Two copies of rows-per-page + hidden-pagination wiring to maintain
- Drifts from the protected `usePagination.bar` / `usePagination.hidden` contract over time

Recommendation:
Extract `admin-edit-request-list-pagination.tsx` (feature-level) that accepts `{ page, pageSize, setPage, setPageSize, total }` and renders hidden table pagination + `PaginationBar`, or extend the page hook return to pass through `bar`/`hidden` without duplicating markup in both tables.

**PR comment (line 67):**  
**Medium:** Both admin list tables hand-build hidden pagination + `PaginationBar` props — please extract a shared feature helper so the footer contract lives in one place.

**Status:** Open

---

---
Edit-request detail loads each source video with a separate API call

Risk Level: HIGH  
File Path: src/pages/editRequest/hooks/useEditRequestDetail.ts  
Lines: 40-50

Description:
Detail hook now loads via `loadEditRequestDetailWithOutputs` and maps `backendRequest.sourceVideos` with `mapEditRequestSourceVideoSummaries` — no per-video `getVideo` fan-out.

**Re-verification (@ `3ae937f6`):** N+1 video fetch removed; backend embeds summaries.

**Status:** ✅ Fixed

---

---
Admin edit-request lists bypass `useServerTableControls` contract

Risk Level: HIGH  
File Path: src/pages/adminEditRequest/index.tsx  
Lines: 100-116

Description:
**Contract / Global consistency.** Page now uses `useServerTableControls` with `extraFilterState` (tab, filters, date range) and external `searchControls` from `useAdminEditRequestListControls` for URL-synced EDR search. `filterKey` resets page on tab/filter/search changes.

**Re-verification (@ `3ae937f6`):** Hand-rolled `useState` pagination removed.

**Status:** ✅ Fixed

---

---
Crew dashboard and insights tabs stale on socket events

Risk Level: MEDIUM  
File Path: src/utils/edit-request-realtime.utils.ts  
Lines: 269-338

Description:
`refetchActiveEditRequestSurfaces` refetches `crewDashboardQueryKey`, and `refetchAdminEditRequestListSurface` invalidates/refetches insights assignment-returns and feedbacks query keys on socket events via global `useEditRequestSocketRealtime`.

**Re-verification (@ `3ae937f6`):** Crew dashboard + insights query keys included in sync path.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed: `useServerTableControls` | CRITICAL | Open | src/hooks/use-server-table-controls.ts | 6-40 |
| 2 | Admin list tables duplicate pagination footer wiring | MEDIUM | Open | src/pages/adminEditRequest/components/admin-edit-request-table.tsx | 67-86 |
| 3 | Edit-request detail loads each source video with a separate API call | HIGH | ✅ Fixed | src/pages/editRequest/hooks/useEditRequestDetail.ts | 40-50 |
| 4 | Admin edit-request lists bypass `useServerTableControls` contract | HIGH | ✅ Fixed | src/pages/adminEditRequest/index.tsx | 100-116 |
| 5 | Crew dashboard and insights tabs stale on socket events | MEDIUM | ✅ Fixed | src/utils/edit-request-realtime.utils.ts | 269-338 |

## Positive notes

- **Global consistency (good):** `refetchActiveEditRequestSurfaces` + `useEditRequestSocketRealtime` unify athlete, admin/editor, crew, insights, and dashboard refresh.
- Crew/editor routing and role gating consistent; editor feedback gates on `myLatestFeedback`.
- **Contract (good):** Output delete uses `DestructiveActionConfirmModal`; export uses `fetchAllPaginatedListItems`; insights tables compose `PaginationBar`.
- Activity history uses `ActivityHistoryTimeline` (domain timeline — `AuditLogTable` N/A).
- URL-synced tab/search via `useAdminEditRequestListControls` improves shareable admin list state.

**Merge readiness:** **Not merge-ready** — 1 open Critical (protected hook change) + 1 open Medium on frontend; 1 open High on backend (`validatedQuery`). Prior High/Medium realtime and list-control findings fixed on branch.
