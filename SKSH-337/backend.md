# Backend PR Review — skillshow (`SKSH-337`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `2068566`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-11 (`2068566` — feedback: protected module revert, crew limit cap, search/sort)  
**Scope:** Crew dashboard assigned-events list (`GET /api/v1/edit-requests/crew/assigned-events`) with search/sort + shared crew pagination utils (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 2 (0 Critical, 0 High) — **0 Open**, **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Not modified** ✅ (reverted since initial review) |
| `list-query-aggregation.utils.ts`, audit-log modules, change-stream modules | Not modified |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/event.constants.ts` | Assigned-events sort map, search fields |
| `src/controllers/edit-request.controller.ts` | `getCrewAssignedEvents` handler |
| `src/repositories/event-crew.repository.ts` | `listAssignedEventsForCrewUser` with search/sort aggregate |
| `src/routes/edit-request.routes.ts` | `GET /crew/assigned-events` route |
| `src/services/edit-request-crew-dashboard.service.ts` | `getAssignedEvents`, dashboard limit via shared util |
| `src/services/edit-request-crew-performance-reviews.service.ts` | Delegates to `parseCrewPaginatedListQuery` |
| `src/types/edit-request/crew-query.types.ts` | `CrewAssignedEventsListQuery` (search/sort) |
| `src/types/edit-request/crew-dashboard.types.ts` | Assigned-events DTOs |
| `src/utils/crew-list-query.utils.ts` | **New** shared crew pagination + list-query parsing |
| `src/validation/edit-request.validation.ts` | `crewAssignedEventsQuerySchema` (page/limit/search/sort) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `parseCrewPaginatedListQuery` / `parseCrewAssignedEventsListQuery` — single crew pagination source | ✅ DRY |
| Performance-reviews service migrated to shared util | ✅ Global consistency |
| `runListQueryAggregate` + `buildRegexOrMatch` for assigned events | ✅ Contract |
| Validation + service both use `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100) | ✅ Fixed (#2) |
| Protected `list-query.validation.ts` untouched | ✅ Fixed (#1) |
| Sort allow-list aligned with frontend column keys + `createdAt` | ✅ |

### Positive notes

- **Shared util:** `crew-list-query.utils.ts` centralizes crew pagination caps (`LIST_QUERY_PAGINATION`) and removes duplicate `parsePagination` from dashboard/performance-reviews services.
- **List contract:** Assigned-events endpoint supports `search`, `sortBy`, `sortOrder` with escaped regex search and `CREW_ASSIGNED_EVENTS_LIST_SORT_FIELD_MAP`.
- **Repository:** Joins assignments → events, filters deleted events, projects lifecycle status with default fallback.

---

## GitHub comments

_No open findings — prior comments resolved on branch._

---

## Findings

---
Protected module changed (list-query.validation.ts)

Risk Level: CRITICAL  
**Status:** ✅ Fixed  
File Path: skillshow/src/validation/list-query.validation.ts  
Lines: 34-36

Description:
**Protected module.** Initial review flagged a whitespace-only reformat of `listQueryOptionalLimitField` in the frozen `list-query.validation.ts` module.

Impact:
- Unrelated churn in a shared validation module.

Recommendation:
Revert the `list-query.validation.ts` hunk — crew limit changes belong in `edit-request.validation.ts` only.

**Re-review evidence:** `list-query.validation.ts` no longer appears in `main...HEAD` diff. Import of `LIST_QUERY_PAGINATION` / `LIST_QUERY_DEFAULTS` from that module in `crew-list-query.utils.ts` is read-only consumption.

---

---
Crew query limit validation vs service cap mismatch

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/utils/crew-list-query.utils.ts  
Lines: 12-23

Description:
**Contract.** Initial review flagged Joi validation allowing `limit` up to 100 while services still clamped with `PAGINATION.MAX_LIMIT` (25).

Impact:
- Clients could pass `limit` 26–100, pass validation, and receive at most 25 rows.

Recommendation:
Align service caps with validation via `LIST_QUERY_PAGINATION` constants.

**Re-review evidence:** `parseCrewPaginatedListQuery` and `parseCrewDashboardListLimit` now clamp with `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100). `edit-request-crew-performance-reviews.service.ts` delegates to the shared util.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Protected module changed (list-query.validation.ts) | CRITICAL | ✅ Fixed | skillshow/src/validation/list-query.validation.ts | 34-36 | — |
| 2 | Crew query limit validation vs service cap mismatch | HIGH | ✅ Fixed | skillshow/src/utils/crew-list-query.utils.ts | 12-23 | — |

**Merge readiness:** **No open backend blockers.** Remaining items are on the frontend report (protected-module scope + default page size).
