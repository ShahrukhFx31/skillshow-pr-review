# Backend PR Review — skillshow (`SKSH-260`)

**Repo:** skillshow  
**Branch:** `SKSH-260`  
**Base:** `main...HEAD` @ `8b1b731`  
**Re-reviewed:** 2026-06-11  
**Scope:** Crew performance reviews API (`GET /v1/edit-requests/crew/performance-reviews`); crew dashboard enrichments (reporting manager, `title`, `latestEditorRating`); shared crew list-query pagination helpers  
**Prompts:** `backend-system-prompt.md`

**Files changed (vs `main`):** 11 — performance-reviews service/controller/routes/validation/types, feedback repository, crew-dashboard service, `list-query.validation` helpers, tests

**Findings:** 0 Open — **2 Fixed**

### Protected modules

PR **adds** `listQueryOptionalPageField` / `listQueryOptionalLimitField` to frozen `list-query.validation.ts` (additive exports only; `createListQuerySchema` and `LIST_QUERY_PAGINATION` bounds unchanged). Used by `crewDashboardQuerySchema` and `crewPerformanceReviewsQuerySchema` for DRY limit/page validation.

### Positive notes

- `getCrewPerformanceReviews` and `getCrewDashboard` now read `validatedQuery` with typed `CrewPerformanceReviewsListQuery` / `CrewDashboardListQuery`.
- `crewPerformanceReviewsQuerySchema` uses `listQueryOptionalLimitField()` (max `PAGINATION.MAX_LIMIT` 25) — aligned with `parsePagination` in the service.
- Actor scoping (`shouldScopeAdminEditRequestsToAssignedEditor` + `actorId === userId`) prevents cross-user review reads.
- Service tests cover summary, forbidden userId, and feedback row mapping.

---

## GitHub comments (Fixed)

### 1. `src/controllers/edit-request.controller.ts` line 169

**Fixed:** `getCrewPerformanceReviews` reads `validatedQuery` as `CrewPerformanceReviewsListQuery` (lines 169–175). `getCrewDashboard` updated the same way (lines 145–150).

### 2. `src/validation/edit-request.validation.ts` line 338

**Fixed:** `crewPerformanceReviewsQuerySchema` uses `listQueryOptionalPageField()` and `listQueryOptionalLimitField()` (max 25 via `PAGINATION.MAX_LIMIT`), matching service `parsePagination`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Controller reads `req.query` instead of `validatedQuery` | HIGH | ✅ Fixed | src/controllers/edit-request.controller.ts | 169-175 |
| 2 | `limit` schema max disagrees with service cap | HIGH | ✅ Fixed | src/validation/edit-request.validation.ts | 335-339 |

**Merge readiness:** **Approve for merge** — no open Critical/High findings.
