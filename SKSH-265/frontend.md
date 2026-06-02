# Frontend PR Review — skillshow-admin-ui (`SKSH-265`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Scope:** Crew dashboard, editor feedback, admin insights tabs, internal revision UI, edit-request detail (Critical, High, Medium only)  
**Findings:** 3 (0 Critical, 1 High, 2 Medium)

> **Scope note:** Pagination / list-query performance findings are omitted per review request.

---

---
Edit-request detail loads each source video with a separate API call

Risk Level: HIGH
File Path: src/pages/editRequest/hooks/useEditRequestDetail.ts
Lines: 63-87

Description:
`useEditRequestDetail` resolves `requestedVideoIds` then runs `Promise.all(videoIds.map((vid) => getVideo(vid)))`. The backend allows up to 20 videos per request (`videos.max(20)`), so a single detail view can trigger up to 20 parallel `GET /videos/:id` calls on every load and refetch.

Impact:
- Slow detail page load and extra server load for multi-video requests
- Mobile/unstable networks see compounded latency and failure risk (one failed video fetch affects the batch)

Recommendation:
Add a batch videos-by-ids API (or include trimmed video metadata on `GET /edit-requests/:id`) and fetch once. Until then, consider limiting concurrency or reusing list payload fields if already available on the edit-request response.

**PR comment (line 68):**  
**High:** Detail hook fans out one `getVideo` per source file (up to 20) — can we batch-fetch or embed video summaries on the edit-request detail response to avoid N parallel calls per page view?

---

---
Crew dashboard does not refresh on edit-request realtime events

Risk Level: MEDIUM
File Path: src/pages/dashboard/crew/dashboard/hooks/use-crew-dashboard.ts
Lines: 15-34

Description:
Crew home uses `useCrewDashboard` with a 30s `staleTime` but does not subscribe to `useAdminEditRequestSocket` / `refetchAdminEditRequestQueries` or invalidate `crewDashboardQueryKey`. After completing work, returning assignment, or receiving socket updates on other admin pages, KPI cards and “recent requests” can stay stale until manual refresh or staleTime expiry.

Impact:
- Crew sees outdated assigned/completed counts and recent list after actions elsewhere in the app
- Average rating card may lag until refetch even after new athlete feedback

Recommendation:
On crew dashboard (or shared crew layout), listen for `EDIT_REQUEST_UPDATED` / `EDIT_REQUEST_CREATED` and `refetchQueries({ queryKey: crewDashboardQueryKey })`, matching the admin list socket pattern.

**PR comment (line 21):**  
**Medium:** Crew dashboard query is not invalidated on edit-request socket events — please refetch `crewDashboardQueryKey` when assignments or completions change so KPIs/recent rows update without waiting 30s.

---

---
Admin insights tabs omit socket-driven cache invalidation

Risk Level: MEDIUM
File Path: src/pages/adminEditRequest/index.tsx
Lines: 55, 156-166

Description:
This PR adds Assignment Returns / Feedback `useQuery` hooks with `ADMIN_EDIT_REQUEST_INSIGHTS_LIST_QUERY_KEY` (lines 156–166) and keeps `useAdminEditRequestSocket()` (line 55), but socket-driven refetch still only hits the main list via `refetchAdminEditRequestQueries` in `src/utils/edit-request-realtime.utils.ts` — that util is **unchanged in this PR** (not visible on the GitHub diff).

Impact:
- Full admins on insights tabs see stale assignment-return or feedback rows after crew returns or new athlete ratings until manual navigation or a new fetch

Recommendation:
When wiring the new insights tabs, extend `refetchAdminEditRequestQueries` in `src/utils/edit-request-realtime.utils.ts` (or refetch from this page’s socket handler) to include:

```typescript
await queryClient.refetchQueries({ queryKey: ADMIN_EDIT_REQUEST_INSIGHTS_LIST_QUERY_KEY.assignmentReturns, type: "active" });
await queryClient.refetchQueries({ queryKey: ADMIN_EDIT_REQUEST_INSIGHTS_LIST_QUERY_KEY.feedbacks, type: "active" });
```

Query keys are defined in `src/pages/adminEditRequest/constants/admin-edit-request.constants.ts` (lines 35–38, in PR).

**PR comment (line 156):**  
**Medium:** These insights queries are not refetched on `EDIT_REQUEST_UPDATED` — please extend `refetchAdminEditRequestQueries` (in `src/utils/edit-request-realtime.utils.ts`) to include `ADMIN_EDIT_REQUEST_INSIGHTS_LIST_QUERY_KEY` so Assignment Returns / Feedback stay in sync after crew/athlete actions.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Edit-request detail loads each source video with a separate API call | HIGH | Open | src/pages/editRequest/hooks/useEditRequestDetail.ts | 63-87 |
| 2 | Crew dashboard does not refresh on edit-request realtime events | MEDIUM | Open | src/pages/dashboard/crew/dashboard/hooks/use-crew-dashboard.ts | 15-34 |
| 3 | Admin insights tabs omit socket-driven cache invalidation | MEDIUM | Open | src/pages/adminEditRequest/index.tsx | 55, 156-166 |

**Positive notes:** Crew/editor dashboard routing and role gating are consistent; editor feedback modal and utils cleanly gate on `myLatestFeedback`; insights tables reuse column hooks; internal-revision and crew-return flows colocate constants and align with backend enums; admin list URL sync for insights tabs is handled in `useAdminEditRequestListControls`.
