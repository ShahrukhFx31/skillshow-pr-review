# Backend PR Review — skillshow (`SKSH-337`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `e03c28f`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-16 (`e03c28f`)  
**Scope:** Crew dashboard assigned-events list (`GET /api/v1/edit-requests/crew/assigned-events`) with search/sort + crew pagination parsing (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules / Security)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 2 (0 Critical, 0 High) — **0 Open**, **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Not modified** |
| `list-query-aggregation.utils.ts` | **Not modified** (prior edit reverted; helpers moved to `edit-request-pagination.utils.ts`) |
| Audit-log modules, change-stream modules | Not modified |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| Route auth: `adminRead` (`authorizeAdminEditRequest`) on new endpoint | ✅ Matches sibling `/crew/dashboard` |
| Service auth: `assertCrewDashboardActor` (crew/editor role gate) | ✅ Defense in depth |
| **IDOR:** list scoped to `actorId` from JWT (`crewUser: actorObjectId`) | ✅ No cross-user `userId` param |
| Search regex escaped via `buildRegexOrMatch` | ✅ |
| No commented/weakened `authorize`, upload, or export paths touched | ✅ |
| No auth bypass regressions introduced in diff | ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/event.constants.ts` | Assigned-events search/sort allow-list constants |
| `src/controllers/edit-request.controller.ts` | `getCrewAssignedEvents` handler |
| `src/repositories/event-crew.repository.ts` | Assigned-events aggregate with search/sort |
| `src/routes/edit-request.routes.ts` | `GET /crew/assigned-events` route |
| `src/services/edit-request-crew-dashboard.service.ts` | Assigned-events response + `parsePageLimitOnly` |
| `src/services/edit-request-crew-performance-reviews.service.ts` | Uses shared `parsePageLimitPagination` |
| `src/types/edit-request/crew-query.types.ts` | Assigned-events query type |
| `src/types/edit-request/crew-dashboard.types.ts` | Assigned-events DTOs |
| `src/utils/edit-request-pagination.utils.ts` | **New** ticket-scoped `parsePageLimit*` helpers |
| `src/validation/edit-request.validation.ts` | `crewAssignedEventsQuerySchema` + crew limit max 100 |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Controller → service → repository layering | ✅ |
| `runListQueryAggregate` + `buildRegexOrMatch` + `parsePageLimitListQuery` | ✅ |
| Crew auth via `assertCrewDashboardActor` | ✅ |
| Validation + service caps both use `LIST_QUERY_PAGINATION` (max 100) | ✅ Fixed (#2) |
| Pagination helpers colocated in `edit-request-pagination.utils.ts` (not protected module) | ✅ Fixed (#1) |
| Performance-reviews service migrated off duplicate `parsePagination` | ✅ DRY |
| Sort allow-list matches frontend column keys + `createdAt` default | ✅ |

### Positive notes

- **Protected-module fix:** `parsePageLimitPagination` / `parsePageLimitOnly` / `parsePageLimitListQuery` now live in `edit-request-pagination.utils.ts`, keeping `list-query-aggregation.utils.ts` untouched.
- **List features:** Assigned-events endpoint supports debounced search (`eventDoc.eventName`) and server sort on event, dates, and status.
- **Repository:** Joins assignments → events, filters deleted events, projects lifecycle status with default fallback.

---

## GitHub comments

_No open findings._

---

## Findings

---
Protected module changed (`list-query-aggregation.utils.ts`)

Risk Level: CRITICAL  
**Status:** ✅ Fixed  
File Path: skillshow/src/utils/list-query-aggregation.utils.ts  
Lines: 35-74

**Re-review evidence:** `list-query-aggregation.utils.ts` is no longer in the PR diff. Pagination parsing was moved to ticket-scoped `src/utils/edit-request-pagination.utils.ts`.

---

---
Crew query limit validation vs service cap mismatch

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/validation/edit-request.validation.ts  
Lines: 334-356

**Re-review evidence:** Crew services/repos now use `parsePageLimitPagination` / `parsePageLimitListQuery` / `parsePageLimitOnly` from `edit-request-pagination.utils.ts`, clamping to `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100) — aligned with Joi validation.

```17:27:skillshow/src/utils/edit-request-pagination.utils.ts
export function parsePageLimitPagination(
  query: PageLimitQuery = {},
  defaultLimit: number = LIST_QUERY_PAGINATION.DEFAULT_PAGE_SIZE
) {
  const page = Math.max(1, query.page ?? LIST_QUERY_PAGINATION.DEFAULT_PAGE);
  const limit = Math.min(
    LIST_QUERY_PAGINATION.MAX_PAGE_SIZE,
    Math.max(1, query.limit ?? defaultLimit)
  );

  return { page, limit, skip: listPaginationSkip(page, limit) };
}
```

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Protected module changed (`list-query-aggregation.utils.ts`) | CRITICAL | ✅ Fixed | skillshow/src/utils/edit-request-pagination.utils.ts | 1-55 | — |
| 2 | Crew query limit validation vs service cap mismatch | HIGH | ✅ Fixed | skillshow/src/validation/edit-request.validation.ts | 334-356 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High blockers.
