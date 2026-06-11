# Frontend PR Review — skillshow-admin-ui (`SKSH-260`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-260`  
**Base:** `main...HEAD` @ `d2236ca1`  
**Reviewed:** 2026-06-11  
**Scope:** Wire crew **Performance Reviews** to `GET /v1/edit-requests/crew/performance-reviews`; crew dashboard UX (reporting manager bar, rating column, route unify to `/dashboard`); remove mock feedback data  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 25 — performance-reviews hook/service/types, `CrewPerformanceReviews`, crew dashboard table/columns/KPI, router + auth redirects, realtime invalidation

**Findings:** 1 Open — **1 Accepted** (0 Critical, 0 High, 1 Medium Open)

### Protected modules

No changes to frozen `use-server-table-controls`, `use-pagination`, `pagination-bar`, `table-sort`, or audit-log modules. Crew edit-requests table correctly uses `usePagination` + `PaginationBar`.

### Positive notes

- Mock `CREW_PERFORMANCE_FEEDBACK_ROWS` / `CREW_PERFORMANCE_RATING_SUMMARY` removed; account tab now loads live API data with skeleton loading.
- `useCrewPerformanceReviews` includes `userId` and `page` in `queryKey`; `enabled: Boolean(userId)` guards unauthenticated fetch.
- `invalidateCrewDashboardQueries` also invalidates `crew-performance-reviews` on socket events — aligned with dashboard KPI refresh.
- Role-based `/dashboard` entry (`pages/dashboard/index.tsx`) unifies crew with coach/parent/super-admin; `Editor` maps to `CrewDashboard`.
- Dashboard recent-requests table reuses `CrewRatingStars`; `mapCrewDashboardEditRequests` maps new `title` / `latestEditorRating` fields cleanly.
- Comment truncation + `PerformanceReviewCommentModal` keeps wide feedback readable without table overflow.

---

## GitHub comments (Accepted — intentional)

### 1. `src/router/index.tsx` line 319

**Accepted (2026-06-11):** No legacy `/dashboard/crew` redirect — crew lands on `/dashboard` via role-based `Dashboard` router; old path not maintained.

---

## GitHub comments (Open findings)

### 1. `src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx` line 15

**PR comment (line 15):** `CREW_PERFORMANCE_REVIEWS_PAGE_SIZE = 10` is duplicated in `use-crew-performance-reviews.ts` line 6 (API `limit`) and here line 144 (`pagination.pageSize`). Export once from `performance-reviews.constants.ts` and import in both files so page size cannot drift.

---

---
Legacy `/dashboard/crew` route removed without redirect

Risk Level: MEDIUM  
File Path: src/router/index.tsx  
Lines: 313-320

Description:
**Global consistency.** On `main`, crew landed at `path: "dashboard/crew"`. This PR registers `path: "dashboard"` with the role-resolving `Dashboard` component and deletes `dashboard/crew` entirely.

Impact:
- Bookmarked `/dashboard/crew` links would 404 after deploy.

Recommendation:
Add a redirect route — **not required per product decision.**

**Re-review:** **Accepted** — No legacy route maintenance; crew uses `/dashboard` only.

---

---
Duplicate `CREW_PERFORMANCE_REVIEWS_PAGE_SIZE` constant

Risk Level: MEDIUM  
File Path: src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx  
Lines: 15, 144 (also `hooks/use-crew-performance-reviews.ts` line 6)

Description:
**DRY.** Page size `10` is defined independently in `CrewPerformanceReviews.tsx` and `use-crew-performance-reviews.ts`. The hook sends `limit` to the API; the table passes `pageSize` to Ant pagination — they must stay identical.

Impact:
- A one-sided edit yields wrong page counts or partial pages (API returns 10 rows but UI paginates as another size).
- Violates single-source-of-truth for a contract param tied to backend pagination.

Recommendation:
Add to `constants/performance-reviews.constants.ts`:

```typescript
export const CREW_PERFORMANCE_REVIEWS_PAGE_SIZE = 10;
```

Import in both the hook and `CrewPerformanceReviews`.

**PR comment (line 15):** **DRY:** `CREW_PERFORMANCE_REVIEWS_PAGE_SIZE` is duplicated here (line 15), at line 144, and in `use-crew-performance-reviews.ts` line 6 — move to `performance-reviews.constants.ts` so API `limit` and table `pageSize` stay in sync.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Legacy `/dashboard/crew` route removed without redirect | MEDIUM | Accepted | src/router/index.tsx | 313-320 |
| 2 | Duplicate `CREW_PERFORMANCE_REVIEWS_PAGE_SIZE` constant | MEDIUM | Open | src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx | 15, 144 |

**Merge readiness:** **Approve with minor fix** — no Critical/High frontend blockers; 1 Medium DRY item (page-size constant). Coordinate with backend: align `limit` cap (backend finding #2) with the shared page-size constant.
