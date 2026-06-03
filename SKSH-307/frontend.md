# Frontend PR Review — skillshow-admin-ui (`SKSH-307`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-307`  
**Base:** `main...HEAD`  
**Commits:** `1f3a2fe6` (fix: coach media pool), merges from `main`  
**Scope:** Media Pool tab on parent/coach connections, paginated media-pool API clients, parent dashboard “View all” + linked-athlete video navigation with `relationId` (Critical, High & Medium)

---

## Overview

Adds `getCoachMediaPool` / `getParentMediaPool`, a `MediaPoolPanel` with pagination on My Athlete / My Roster (and account connections), Segmented toggle between directory and Media Pool (`?section=media-pool`), shared `mapDashboardFeedItemsToRecentVideos` and `navigateToLinkedAthleteVideoDetails`, and parent dashboard deep links to the media pool with correct back-navigation on linked video details.

---

---
Media Pool query not refreshed on socket dashboard events

Risk Level: MEDIUM  
File Path: src/pages/user/account/connections/components/MediaPoolPanel.tsx  
Lines: 26-32

Description:
`MediaPoolPanel` uses `useQuery` with `queryKey: ["media-pool", mode, page, pageSize]`. `useParentDashboardRealtime` invalidates only `parentDashboardQueryKey` and `coachDashboardQueryKey` on `ATHLETE_RELATION_EVENTS` and `SOCKET_EVENTS.VIDEO_LINKED_ATHLETE_UPDATED`. Users on the Media Pool tab will keep stale cards until manual refresh, tab remount, or window refocus (default React Query behavior).

Impact:
- New linked-athlete uploads or relation changes do not update the Media Pool grid while the tab is open.
- Inconsistent with parent dashboard athlete feed, which refreshes via the same socket events.

Recommendation:
Extend `useParentDashboardRealtime` (or a small sibling hook) to also invalidate the media-pool key prefix, e.g.:

```typescript
void queryClient.invalidateQueries({ queryKey: ["media-pool"] });
```

Alternatively colocate a `mediaPoolQueryKey` in `@/constants/queryKeys` and invalidate that from the same socket handlers.

**PR comment (line 31):** **Medium:** Socket handlers refresh dashboard queries but not `["media-pool", …]`, so the Media Pool tab can stay stale after uploads or link changes. Please invalidate the media-pool query key from `useParentDashboardRealtime` (or equivalent) alongside the dashboard keys.

---

## Findings (Critical / High)

No **Critical** or **High** issues identified in scope.

---

## Positive notes

- **Navigation:** `navigateToLinkedAthleteVideoDetails` centralizes linked-athlete details + breadcrumb state for dashboard feed, Media Pool, and existing vault flows.
- **Composition:** `MediaPoolPanel` is colocated under connections `components/`; API types in `@/types/dashboard.types`.
- **UX:** Deep-link `?relationId=` forces Athlete directory (not Media Pool) so highlighted rows remain visible; parent “View all” routes to `?section=media-pool`.
- **Pagination:** `TeamTablePaginationBar` resets page to 1 on page-size change.
- **Feed mapping:** `mapDashboardFeedItemsToRecentVideos` unifies `childNameTag` / `athleteNameTag` and passes `relationId` through to `RecentVideos`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Media Pool query not refreshed on socket dashboard events | MEDIUM | Open | src/pages/user/account/connections/components/MediaPoolPanel.tsx | 26-32 |

**Merge readiness:** No open Critical/High blockers; one Medium follow-up (socket invalidation for Media Pool) before treating frontend review as fully closed.
