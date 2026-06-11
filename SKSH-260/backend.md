# Backend PR Review — skillshow (`SKSH-260`)

**Repo:** skillshow  
**Branch:** `SKSH-260`  
**Base:** `main...HEAD` @ `15f23c0`  
**Re-reviewed:** 2026-06-11 (no new commits since initial review)  
**Scope:** Crew performance reviews API (`GET /v1/edit-requests/crew/performance-reviews`); crew dashboard enrichments (reporting manager, `title`, `latestEditorRating` on recent rows)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 9 — performance-reviews service/controller/routes/validation/types, feedback repository aggregations, crew-dashboard service, tests

**Findings:** 2 Open (0 Critical, 2 High)

### Protected modules

No changes to frozen `list-query.validation.ts`, `list-query-aggregation.utils.ts`, change-stream, or audit-log modules.

### Positive notes

- New route uses Joi `crewPerformanceReviewsQuerySchema` + `validate(..., "query")` and `adminRead` — consistent with crew dashboard sibling.
- Actor scoping (`shouldScopeAdminEditRequestsToAssignedEditor` + `actorId === userId`) prevents cross-user review reads.
- Feedback list uses existing `listPaginated`; context hydration batches `findInsightsContextByIds` + `findLeanProfileDisplayByIds`.
- `ratingBreakdownForEditor` fills all star buckets (5→1) for stable UI bars.
- Service tests cover summary shape, forbidden userId, and feedback row mapping.

---

## GitHub comments (Open findings)

### 1. `src/controllers/edit-request.controller.ts` line 166

**PR comment (line 166):** `getCrewPerformanceReviews` still passes `req.query` into the service even though `validate(crewPerformanceReviewsQuerySchema, "query")` writes coerced values to `req.validatedQuery` (see `validate.middleware.ts` line 11). Use `(req as ValidatedQueryRequest).validatedQuery` like `app-user.controller.ts` (line 36) so `page`/`limit` are numbers and `userId` is the Joi-validated ObjectId string.

### 2. `src/validation/edit-request.validation.ts` line 333

**PR comment (line 333):** `crewPerformanceReviewsQuerySchema` still allows `limit` up to **100** (`.max(100)` on this line), but `EditRequestCrewPerformanceReviewsService.parsePagination` (`edit-request-crew-performance-reviews.service.ts` lines 21–27) caps at `PAGINATION.MAX_LIMIT` (**25**). Align with `crewDashboardQuerySchema` (line 327, `.max(25)`).

---

---
Controller reads `req.query` instead of `validatedQuery`

Risk Level: HIGH  
File Path: src/controllers/edit-request.controller.ts  
Lines: 160-167

Description:
**Contract.** The route wires `validate(crewPerformanceReviewsQuerySchema, "query")`, which stores coerced query params on `req.validatedQuery`. `getCrewPerformanceReviews` forwards `req.query` to the service.

Impact:
- Typed/coerced `page` and `limit` from Joi may not reach the service; pagination relies on ad-hoc `Number()` coercion in `parsePagination`.
- Future schema changes on `validatedQuery` will not apply to this handler.
- Inconsistent with list controllers that spread `validatedQuery` into services.

Recommendation:
Read validated query in the controller and pass it through:

```typescript
import type { ValidatedQueryRequest } from "../types/common";

async getCrewPerformanceReviews(req: AuthenticatedRequest, res: Response) {
  const actorId = req.user!.userId;
  const query = (req as ValidatedQueryRequest).validatedQuery ?? {};
  const result =
    await editRequestCrewPerformanceReviewsService.getPerformanceReviews(
      actorId,
      query
    );
  // ...
}
```

**Re-review:** Still **Open** at `15f23c0` — line 166 still passes `req.query`.

---

---
`limit` schema max (100) disagrees with service cap (25)

Risk Level: HIGH  
File Path: src/validation/edit-request.validation.ts  
Lines: 330-334

Description:
**Contract / Global consistency.** `crewPerformanceReviewsQuerySchema` allows `limit` up to 100, while `parsePagination` clamps to `PAGINATION.MAX_LIMIT` (25). Sibling `crewDashboardQuerySchema` correctly uses `.max(25)`.

Impact:
- API accepts `limit=50` or `limit=100` but silently returns at most 25 rows.
- Response `pagination.limit` echoes the requested value while the query used 25 — clients cannot trust pagination metadata.

Recommendation:
Align validation with the service:

```typescript
limit: Joi.number().integer().min(1).max(25).optional().label("limit"),
```

**Re-review:** Still **Open** at `15f23c0` — line 333 still `.max(100)`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Controller reads `req.query` instead of `validatedQuery` | HIGH | Open | src/controllers/edit-request.controller.ts | 160-167 |
| 2 | `limit` schema max (100) disagrees with service cap (25) | HIGH | Open | src/validation/edit-request.validation.ts | 330-334 |

**Merge readiness:** **Request changes** — 2 open High (query contract + pagination cap mismatch). No Critical findings; auth scoping, repository aggregations, and service tests are otherwise sound.
