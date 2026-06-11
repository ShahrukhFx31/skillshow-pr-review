# Backend PR Review — skillshow (`SKSH-260`)

**Repo:** skillshow  
**Branch:** `SKSH-260`  
**Base:** `main...HEAD` @ `15f23c0`  
**Reviewed:** 2026-06-11  
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

**PR comment (line 166):** `getCrewPerformanceReviews` passes `req.query` into the service even though `validate(crewPerformanceReviewsQuerySchema, "query")` writes coerced values to `req.validatedQuery` (see `validate.middleware.ts` line 11). Use `(req as ValidatedQueryRequest).validatedQuery` like `app-user.controller.ts` (line 36) so `page`/`limit` are numbers and `userId` is the Joi-validated ObjectId string — not raw query strings that bypass the middleware contract.

### 2. `src/validation/edit-request.validation.ts` line 333

**PR comment (line 333):** `crewPerformanceReviewsQuerySchema` allows `limit` up to **100** (`.max(100)` on this line), but `EditRequestCrewPerformanceReviewsService.parsePagination` (`edit-request-crew-performance-reviews.service.ts` lines 21–27) caps at `PAGINATION.MAX_LIMIT` (**25**). Align the schema with the service (`.max(25)`, matching `crewDashboardQuerySchema` line 327) — otherwise clients can pass `limit=50`, get a 200 with only 25 rows, and `pagination.limit` still reports 50.

---

---
Controller reads `req.query` instead of `validatedQuery`

Risk Level: HIGH  
File Path: src/controllers/edit-request.controller.ts  
Lines: 160-167

Description:
**Contract.** The route wires `validate(crewPerformanceReviewsQuerySchema, "query")`, which stores coerced query params on `req.validatedQuery` (`validate.middleware.ts` documents that controllers must not read list/query params from raw `req.query`). `getCrewPerformanceReviews` forwards `req.query` to the service, duplicating the pre-existing `getCrewDashboard` pattern but violating the established validated-query contract for new code.

Impact:
- Typed/coerced `page` and `limit` from Joi may not reach the service; pagination relies on ad-hoc `Number()` coercion in `parsePagination`.
- Future schema changes (defaults, transforms) on `validatedQuery` will not apply to this handler.
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

Drop the redundant manual `userId` string guard in the service once `validatedQuery` is the sole source (Joi already requires `userId`).

**PR comment (line 166):** This route validates query via Joi but still passes `req.query` to the service — please use `req.validatedQuery` so coerced `page`/`limit`/`userId` match the middleware contract (same pattern as `app-user.controller.ts` line 36).

---

---
`limit` schema max (100) disagrees with service cap (25)

Risk Level: HIGH  
File Path: src/validation/edit-request.validation.ts  
Lines: 330-334

Description:
**Contract / Global consistency.** `crewPerformanceReviewsQuerySchema` allows `limit` up to 100, while `parsePagination` in `edit-request-crew-performance-reviews.service.ts` clamps to `PAGINATION.MAX_LIMIT` (25). Sibling `crewDashboardQuerySchema` correctly uses `.max(25)`.

Impact:
- API accepts `limit=50` or `limit=100` but silently returns at most 25 rows.
- Response `pagination.limit` echoes the requested value (e.g. 50) while the query used 25 — clients cannot trust pagination metadata.
- Divergent caps between validation and service will drift if `PAGINATION.MAX_LIMIT` changes.

Recommendation:
Align validation with the service (preferred — matches crew dashboard):

```typescript
export const crewPerformanceReviewsQuerySchema = Joi.object({
  userId: objectId.required().label("userId"),
  page: Joi.number().integer().min(1).optional().label("page"),
  limit: Joi.number().integer().min(1).max(25).optional().label("limit"),
}).options(opts);
```

Alternatively export a shared `CREW_PERFORMANCE_REVIEWS_MAX_LIMIT` used by both schema and `parsePagination`.

**PR comment (line 333):** Joi allows `limit` up to 100 (`.max(100)` here) but the service caps at `PAGINATION.MAX_LIMIT` (25) — please align with `crewDashboardQuerySchema` (line 327, `.max(25)`) so validation and `pagination.limit` stay truthful.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Controller reads `req.query` instead of `validatedQuery` | HIGH | Open | src/controllers/edit-request.controller.ts | 160-167 |
| 2 | `limit` schema max (100) disagrees with service cap (25) | HIGH | Open | src/validation/edit-request.validation.ts | 330-334 |

**Merge readiness:** **Request changes** — 2 open High (query contract + pagination cap mismatch). No Critical findings; auth scoping, repository aggregations, and service tests are otherwise sound.
