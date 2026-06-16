# Backend PR Review — skillshow (`SKSH-337`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `8f2612b`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-12 (`8f2612b`)  
**Scope:** Crew dashboard assigned-events list (`GET /api/v1/edit-requests/crew/assigned-events`) with search/sort + shared crew `page`/`limit` parsing (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 2 (0 Critical, 1 High) — **0 Open**, **1 Fixed**, **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Not modified** (prior whitespace change reverted — #1 Fixed) |
| `list-query-aggregation.utils.ts` | **Modified** — `parsePageLimitPagination` / `parsePageLimitOnly` / `parsePageLimitListQuery` added (**Accepted** — centralizes crew `page`/`limit` parsing per list-query contract; removes duplicate service `parsePagination`) |
| Audit-log modules, change-stream modules | Not modified |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/event.constants.ts` | Assigned-events sort/search field maps |
| `src/controllers/edit-request.controller.ts` | `getCrewAssignedEvents` handler |
| `src/repositories/event-crew.repository.ts` | `listAssignedEventsForCrewUser` with search/sort aggregate |
| `src/routes/edit-request.routes.ts` | `GET /crew/assigned-events` route |
| `src/services/edit-request-crew-dashboard.service.ts` | `getAssignedEvents`; `parsePageLimitOnly` for dashboard limit |
| `src/services/edit-request-crew-performance-reviews.service.ts` | Uses shared `parsePageLimitPagination` |
| `src/types/edit-request/crew-query.types.ts` | `CrewAssignedEventsListQuery` with search/sort |
| `src/types/edit-request/crew-dashboard.types.ts` | Assigned-events DTOs |
| `src/utils/list-query-aggregation.utils.ts` | Shared `parsePageLimit*` helpers |
| `src/validation/edit-request.validation.ts` | `crewAssignedEventsQuerySchema` + crew limit max 100 |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Controller → service → repository layering | ✅ |
| `runListQueryAggregate` + `buildRegexOrMatch` + `parsePageLimitListQuery` | ✅ |
| Crew auth via `assertCrewDashboardActor` | ✅ |
| Validation + service caps both use `LIST_QUERY_PAGINATION` (max 100) | ✅ Fixed (#2) |
| Performance-reviews service migrated off duplicate `parsePagination` | ✅ DRY |
| `list-query.validation.ts` untouched | ✅ Fixed (#1) |
| Sort allow-list (`CREW_ASSIGNED_EVENTS_LIST_SORT_BY_VALUES`) matches frontend column keys + `createdAt` | ✅ |

### Positive notes

- **Shared pagination helpers:** `parsePageLimitPagination` / `parsePageLimitListQuery` align crew endpoints on `LIST_QUERY_PAGINATION` bounds and eliminate per-service clamp drift.
- **List features:** Assigned-events endpoint supports debounced search (`eventDoc.eventName`) and server sort on event, dates, and status.
- **Repository:** Joins assignments → events, filters deleted events, projects lifecycle status with default fallback.

---

## GitHub comments

_No open findings._

---

## Findings

---
Protected module changed (list-query.validation.ts)

Risk Level: CRITICAL  
**Status:** ✅ Fixed  
File Path: skillshow/src/validation/list-query.validation.ts  
Lines: 34-36

**Re-review evidence:** `list-query.validation.ts` is no longer in the PR diff; whitespace-only reformat reverted.

---

---
Crew query limit validation vs service cap mismatch

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/validation/edit-request.validation.ts  
Lines: 329-343

**Re-review evidence:** Crew services/repos now use `parsePageLimitPagination` / `parsePageLimitListQuery` / `parsePageLimitOnly` from `list-query-aggregation.utils.ts`, clamping to `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100) — aligned with Joi validation.

```35:46:skillshow/src/utils/list-query-aggregation.utils.ts
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
| 1 | Protected module changed (list-query.validation.ts) | CRITICAL | ✅ Fixed | skillshow/src/validation/list-query.validation.ts | 34-36 | — |
| 2 | Crew query limit validation vs service cap mismatch | HIGH | ✅ Fixed | skillshow/src/validation/edit-request.validation.ts | 329-343 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High blockers.
