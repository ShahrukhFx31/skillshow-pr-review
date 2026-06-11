# Backend PR Review — skillshow (`SKSH-194`)

**Repo:** skillshow  
**Branch:** `SKSH-194-1`  
**Base:** `main...HEAD` @ `5ab5bce`  
**Scope:** Crew user work dashboard API — summary, paginated edit-request and event lists for assigned editor/crew (`GET /v1/crew-users/:id/work`, `/work/edit-requests`, `/work/events`)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 1 in scope (0 Critical, 1 High) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Files reviewed (9 changed)

`crew-user.constants.ts`, `crew-user.controller.ts`, `crew-user.service.ts`, `crew-user.routes.ts`, `crew-user.validation.ts`, `crew-user-work.types.ts`, `edit-request.repository.ts`, `event-crew.repository.ts`, `crew-user.service.test.ts` (mock wiring only).

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| List schemas use `createListQuerySchema` + `validatedQuery` on routes | ✅ |
| Edit-request list uses `listSortSpec` + sort field map constants | ✅ |
| Events list uses `runListQueryAggregate` + `buildRegexOrMatch` | ✅ |
| Edit-request search reuses `escapeRegex`, EDR seq, date parse, user lookup patterns | ✅ |
| Athlete counts via `eventAthleteService.countAthletesByEvents` (matches `event.service.listEvents`) | ✅ |
| Aggregation also `$lookup`s athlete registrations but drops count before DTO mapping | ⚠️ See #1 |
| `listWorkEditRequests` merges `DEFAULT_LIST_QUERY`; `listWorkEvents` relies on Joi defaults only | ✅ Acceptable (both get bounded pagination) |
| New work endpoints covered by tests | ⚠️ Mocks added only; no new assertions (out of High scope) |

---

## GitHub comments (Open findings)

### 1. `src/repositories/event-crew.repository.ts` line 42

**PR comment (line 42):** **High (DRY / KISS):** `CREW_USER_WORK_EVENTS_LOOKUP_STAGES` joins `eventathleteregistrations` and computes `athleteCount`, but `CREW_USER_WORK_EVENTS_LIST_PROJECT` never projects it and `crew-user.service` calls `countAthletesByEvents` anyway. Drop the unused lookup/`$addFields` so each list page avoids an extra join per assignment row.

---

## Findings

---
Unused athlete-registration lookup in crew work events aggregate

Risk Level: HIGH  
File Path: src/repositories/event-crew.repository.ts  
Lines: 42-54

Description:
**DRY / KISS.** `CREW_USER_WORK_EVENTS_LOOKUP_STAGES` performs a `$lookup` on `eventathleteregistrations` and sets `athleteCount` via `$size`, but `CREW_USER_WORK_EVENTS_LIST_PROJECT` does not include `athleteCount` (or the registration array). `crew-user.service.listCrewUserWorkEvents` then calls `eventAthleteService.countAthletesByEvents` — the same pattern as `event.service.listEvents` — to populate athlete counts in `toEventDto`.

Impact:
- Every crew-work events list request pays for a per-row athlete-registration join whose result is discarded
- Extra Mongo load and latency at scale with no behavioral benefit
- Future maintainers may assume pipeline `athleteCount` is authoritative and diverge from `countAthletesByEvents`

Recommendation:
Remove the athlete-registration lookup and `$addFields` from `CREW_USER_WORK_EVENTS_LOOKUP_STAGES`; keep only the event `$lookup` + `$unwind`. Continue using `countAthletesByEvents` in the service for DTO mapping.

```typescript
const CREW_USER_WORK_EVENTS_LOOKUP_STAGES: PipelineStage[] = [
  {
    $lookup: {
      from: "events",
      localField: "event",
      foreignField: "_id",
      as: "event",
    },
  },
  { $unwind: "$event" },
];
```

**PR comment (line 42):** **High (DRY / KISS):** Remove the unused `eventathleteregistrations` lookup — athlete counts already come from `countAthletesByEvents` in the service.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unused athlete-registration lookup in crew work events aggregate | HIGH | Open | src/repositories/event-crew.repository.ts | 42-54 |

### Positive notes

- Work list validation follows `createListQuerySchema` with entity-specific sort maps and edit-request status filter override.
- `listByEditorPaginated` uses `escapeRegex`, `listSortSpec`, and lean reads consistent with `listByUserPaginated`.
- Routes place `/work` paths before `/:crewUserId` to avoid param shadowing.
- Summary endpoint parallelizes counts and average rating with `Promise.all`.

**Merge readiness:** Blocked — 1 open High (remove dead aggregation join before merge).
