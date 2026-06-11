# Backend PR Review — skillshow (`SKSH-337`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `6e30b55`  
**Initial review:** 2026-06-11  
**Scope:** Crew dashboard assigned-events list endpoint (`GET /api/v1/edit-requests/crew/assigned-events`) + crew query limit validation alignment (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 2 (1 Critical, 1 High) — **2 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Modified** — whitespace-only reformat (see #1) |
| `list-query-aggregation.utils.ts`, audit-log modules, change-stream modules | Not modified |

### Files reviewed

| File | Change |
|------|--------|
| `src/controllers/edit-request.controller.ts` | `getCrewAssignedEvents` handler |
| `src/repositories/event-crew.repository.ts` | `listAssignedEventsForCrewUser` aggregate |
| `src/routes/edit-request.routes.ts` | `GET /crew/assigned-events` route |
| `src/services/edit-request-crew-dashboard.service.ts` | `getAssignedEvents`, `parsePagination`, row mapper |
| `src/types/edit-request/crew-dashboard.types.ts` | Assigned-events DTOs |
| `src/validation/edit-request.validation.ts` | `crewAssignedEventsQuerySchema`; limit max → 100 on crew schemas |
| `src/validation/list-query.validation.ts` | Whitespace reformat of `listQueryOptionalLimitField` |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| New endpoint follows controller → service → repository layering | ✅ |
| `runListQueryAggregate` + event `$lookup` reuse established event-crew patterns | ✅ |
| Crew auth via `assertCrewDashboardActor` / `shouldScopeAdminEditRequestsToAssignedEditor` | ✅ |
| `parsePagination` added to dashboard service (duplicate of performance-reviews service) | ⚠️ Accepted — same pattern as sibling crew service |
| Validation `limit` max raised to `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100) on crew schemas | ❌ Service still caps at `PAGINATION.MAX_LIMIT` (25) — see #2 |
| Protected `list-query.validation.ts` touched for unrelated formatting | ❌ See #1 |

### Positive notes

- **Repository design:** `listAssignedEventsForCrewUser` joins assignments → events, filters `eventDoc.isDeleted: false`, and projects lifecycle status with `EVENT_DEFAULT_LIFECYCLE_STATUS` fallback.
- **Response shape:** DTO maps `endDate` → `dueDate` and formats dates via `formatEventDateForDto`, aligned with frontend column labels.
- **Auth:** Route uses `adminRead` + Joi validation + same forbidden-message handling as other crew dashboard endpoints.

---

## GitHub comments

### 1. `src/validation/list-query.validation.ts` line 34

**PR comment (line 34):** **Critical (Protected module):** This PR reformats `listQueryOptionalLimitField` in the frozen `list-query.validation.ts` module. Please revert this whitespace-only change; crew limit updates belong in `edit-request.validation.ts` only.

### 2. `src/validation/edit-request.validation.ts` line 337

**PR comment (line 337):** **High (Contract):** `crewAssignedEventsQuerySchema` (and sibling crew schemas) now allow `limit` up to `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100), but `parsePagination` / `getDashboard` still clamp to `PAGINATION.MAX_LIMIT` (25). Clients can pass `limit=50`, pass validation, and receive only 25 rows with mismatched pagination metadata. Align service caps with validation (use `LIST_QUERY_PAGINATION` constants) or revert validation max to 25.

---

## Findings

---
Protected module changed (list-query.validation.ts)

Risk Level: CRITICAL  
**Status:** Open  
File Path: skillshow/src/validation/list-query.validation.ts  
Lines: 34-36

Description:
**Protected module.** The PR modifies `list-query.validation.ts`, a frozen shared module. The diff only collapses `listQueryOptionalLimitField` from a multi-line Joi chain to a single line — no functional change — but the file is outside SKSH-337 scope.

Impact:
- Unrelated churn in a shared validation module increases merge conflict risk on list-query work.
- Violates the protected-module contract (changes should be scoped to dedicated tickets).

Recommendation:
Revert the `list-query.validation.ts` hunk entirely. Keep all crew limit changes in `edit-request.validation.ts` (already correct location).

**PR comment (`list-query.validation.ts` line 34):** **Critical (Protected module):** This PR reformats `listQueryOptionalLimitField` in the frozen `list-query.validation.ts` module. Please revert this whitespace-only change; crew limit updates belong in `edit-request.validation.ts` only.

---

---
Crew query limit validation vs service cap mismatch

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow/src/validation/edit-request.validation.ts  
Lines: 329-343

Description:
**Contract / Global consistency.** This PR changes `crewDashboardQuerySchema`, `crewAssignedEventsQuerySchema`, and `crewPerformanceReviewsQuerySchema` to use `listQueryOptionalLimitField(LIST_QUERY_PAGINATION.MAX_PAGE_SIZE)` (max **100**). Service-layer pagination in `edit-request-crew-dashboard.service.ts` and `edit-request-crew-performance-reviews.service.ts` still clamps with `PAGINATION.MAX_LIMIT` (**25**):

```52:58:skillshow/src/services/edit-request-crew-dashboard.service.ts
  private parsePagination(query: CrewPaginatedListQuery = {}) {
    const page = Math.max(1, query.page ?? 1);
    const limit = Math.min(
      PAGINATION.MAX_LIMIT,
      Math.max(1, query.limit ?? PAGINATION.DEFAULT_LIMIT)
    );
    return { page, limit };
  }
```

`getDashboard` applies the same `PAGINATION.MAX_LIMIT` clamp at line 258-261.

Impact:
- API accepts `limit` 26–100 but silently returns at most 25 rows; `pagination.limit` in the response reflects the clamped value while clients may believe a higher limit was honored.
- Frontend `PaginationBar` exposes page sizes up to 100 (`PAGE_SIZE_OPTIONS`); selecting 50 or 100 will not match row count (see frontend.md #1).

Recommendation:
Align service caps with validation — replace `PAGINATION.MAX_LIMIT` with `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` in crew `parsePagination` paths (and `getDashboard`), and default to `LIST_QUERY_PAGINATION.DEFAULT_PAGE_SIZE` (10) where appropriate:

```typescript
import { LIST_QUERY_PAGINATION } from "../validation/list-query.validation";

const limit = Math.min(
  LIST_QUERY_PAGINATION.MAX_PAGE_SIZE,
  Math.max(1, query.limit ?? LIST_QUERY_PAGINATION.DEFAULT_PAGE_SIZE)
);
```

Alternatively, if 25 is the intended crew cap, revert validation to `listQueryOptionalLimitField()` (default max 25) instead of raising the Joi max to 100.

**PR comment (`edit-request.validation.ts` line 337):** **High (Contract):** `crewAssignedEventsQuerySchema` (and sibling crew schemas) now allow `limit` up to `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100), but `parsePagination` / `getDashboard` still clamp to `PAGINATION.MAX_LIMIT` (25). Clients can pass `limit=50`, pass validation, and receive only 25 rows with mismatched pagination metadata. Align service caps with validation (use `LIST_QUERY_PAGINATION` constants) or revert validation max to 25.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Protected module changed (list-query.validation.ts) | CRITICAL | Open | skillshow/src/validation/list-query.validation.ts | 34-36 | 34 |
| 2 | Crew query limit validation vs service cap mismatch | HIGH | Open | skillshow/src/validation/edit-request.validation.ts | 329-343 | 337 |

**Merge readiness:** **Not merge-ready** — 1 Critical (protected module) + 1 High (pagination contract). Fix before merge.
