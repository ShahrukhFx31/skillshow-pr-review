# Backend PR Review — skillshow (`SKSH-337`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `017962d`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-16 (`017962d`)  
**Scope:** Crew dashboard assigned-events list (`GET /api/v1/edit-requests/crew/assigned-events`) with search/sort + shared crew pagination parsing (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 2 (1 Critical, 0 High) — **1 Open**, **1 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Not modified** (prior formatting issue remains fixed) |
| `list-query-aggregation.utils.ts` | **Modified** — added shared `parsePageLimit*` helpers (**Open Critical**, protected module edit in this ticket) |
| Audit-log modules, change-stream modules | Not modified |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/event.constants.ts` | Assigned-events search/sort allow-list constants |
| `src/controllers/edit-request.controller.ts` | `getCrewAssignedEvents` handler |
| `src/repositories/event-crew.repository.ts` | Assigned-events aggregate with search/sort |
| `src/routes/edit-request.routes.ts` | `GET /crew/assigned-events` route |
| `src/services/edit-request-crew-dashboard.service.ts` | Assigned-events response + `parsePageLimitOnly` usage |
| `src/services/edit-request-crew-performance-reviews.service.ts` | Migrated to shared `parsePageLimitPagination` |
| `src/types/edit-request/crew-query.types.ts` | Assigned-events query type |
| `src/utils/list-query-aggregation.utils.ts` | Added `parsePageLimitPagination`, `parsePageLimitOnly`, `parsePageLimitListQuery` |
| `src/validation/edit-request.validation.ts` | Crew assigned-events query schema with search/sort defaults |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Controller → service → repository layering | ✅ |
| Event repo list uses `runListQueryAggregate` + shared `buildRegexOrMatch` path | ✅ |
| Crew auth checks retained (`assertCrewDashboardActor`) | ✅ |
| Validation and service cap aligned to `LIST_QUERY_PAGINATION.MAX_PAGE_SIZE` (100) | ✅ Fixed |
| Shared parsing extracted to `list-query-aggregation.utils.ts` | ✅ DRY improvement, but violates protected-module scope in this ticket |

### Positive notes

- **Contract parity:** `crewAssignedEventsQuerySchema` now includes `search` + `sortBy` + `sortOrder` allow-list aligned with `CREW_ASSIGNED_EVENTS_LIST_SORT_BY_VALUES`.
- **Repository correctness:** Assigned-events pipeline filters deleted events and defaults missing lifecycle status via `EVENT_DEFAULT_LIFECYCLE_STATUS`.
- **Cross-endpoint consistency:** Crew performance reviews now reuse the same page/limit parser used by assigned events.

---

## GitHub comments

### 1. `src/utils/list-query-aggregation.utils.ts` line 35

**PR comment (line 35):** **Critical (Protected module):** `list-query-aggregation.utils.ts` is a frozen shared module, but this ticket adds `parsePageLimitPagination`, `parsePageLimitOnly`, and `parsePageLimitListQuery` here. Please move these helpers into a ticket-scoped module (or revert this file) and keep this PR focused on consumer-side wiring only.

---

## Findings

---
Protected module changed (`list-query-aggregation.utils.ts`)

Risk Level: CRITICAL  
**Status:** Open  
File Path: skillshow/src/utils/list-query-aggregation.utils.ts  
Lines: 35-74

Description:
**Protected module.** The PR adds new pagination parsing helpers to `list-query-aggregation.utils.ts`, which is listed as a frozen module in the backend review contract.

Impact:
- Introduces shared-contract surface area inside a ticket that is not explicitly scoped to protected module changes.
- Raises regression risk for other list endpoints that rely on this utility module.

Recommendation:
Revert changes in `list-query-aggregation.utils.ts` for this ticket and colocate page/limit parsing in a ticket-scoped helper under the edit-request domain (service-level or feature util). If shared helper changes are required, move this to a dedicated protected-module ticket and link it.

**PR comment (`list-query-aggregation.utils.ts` line 35):** **Critical (Protected module):** `list-query-aggregation.utils.ts` is a frozen shared module, but this ticket adds `parsePageLimitPagination`, `parsePageLimitOnly`, and `parsePageLimitListQuery` here. Please move these helpers into a ticket-scoped module (or revert this file) and keep this PR focused on consumer-side wiring only.

---

---
Crew query limit validation vs service cap mismatch

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/validation/edit-request.validation.ts  
Lines: 334-356

**Re-review evidence:** Crew services now parse page/limit with `LIST_QUERY_PAGINATION`-based helpers; the previous 25-row cap mismatch is removed.

```334:356:skillshow/src/validation/edit-request.validation.ts
export const crewDashboardQuerySchema = Joi.object({
  limit: listQueryOptionalLimitField(LIST_QUERY_PAGINATION.MAX_PAGE_SIZE),
}).options(opts);

export const crewAssignedEventsQuerySchema = Joi.object({
  page: listQueryOptionalPageField(),
  limit: listQueryOptionalLimitField(LIST_QUERY_PAGINATION.MAX_PAGE_SIZE),
  search: listQueryOptionalStringFilter(),
  sortBy: Joi.string()
    .valid(...CREW_ASSIGNED_EVENTS_LIST_SORT_BY_VALUES)
    .optional()
    .default(LIST_QUERY_DEFAULTS.SORT_BY),
  sortOrder: Joi.string()
    .valid("asc", "desc")
    .optional()
    .default(LIST_QUERY_DEFAULTS.SORT_ORDER),
}).options(opts);
```

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Protected module changed (`list-query-aggregation.utils.ts`) | CRITICAL | Open | skillshow/src/utils/list-query-aggregation.utils.ts | 35-74 | 35 |
| 2 | Crew query limit validation vs service cap mismatch | HIGH | ✅ Fixed | skillshow/src/validation/edit-request.validation.ts | 334-356 | — |

**Merge readiness:** **Not merge-ready** — 1 open Critical blocker (protected module scope).
