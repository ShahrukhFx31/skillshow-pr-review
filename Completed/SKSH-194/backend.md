# Backend PR Review — skillshow (`SKSH-194`)

**Repo:** skillshow  
**Branch:** `SKSH-194-1`  
**Base:** `main...HEAD` @ `a535da7`  
**Re-reviewed:** 2026-06-11 (post `fix: added fix`)  
**Scope:** Crew user work dashboard API — summary, paginated edit-request and event lists for assigned editor/crew (`GET /v1/crew-users/:id/work`, `/work/edit-requests`, `/work/events`)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 1 in scope (0 Critical, 1 High) — **0 Open**, **1 Fixed**

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
| `seq` added to edit-request sort map (aligns with Request ID column) | ✅ |
| Events list uses `runListQueryAggregate` + `buildRegexOrMatch` | ✅ |
| Athlete-registration lookup only when `sortBy === "athletes"` | ✅ Fixed (#1) |
| Athlete DTO counts via `countAthletesByEvents` (matches `event.service.listEvents`) | ✅ |
| Edit-request search reuses `escapeRegex`, EDR seq, date parse, user lookup patterns | ✅ |
| New work endpoints covered by tests | ⚠️ Mocks added only; no new assertions (out of High scope) |

---

## GitHub comments

_No open findings — prior comment resolved in `a535da7`._

### Resolved — `src/repositories/event-crew.repository.ts` line 44

**PR comment (line 44):** ~~**High (DRY / KISS):** Remove always-on `eventathleteregistrations` lookup.~~ **Resolved** — lookup moved to `CREW_USER_WORK_EVENTS_ATHLETE_COUNT_STAGES` and applied only when `query.sortBy === "athletes"`.

---

## Findings

---
Unused athlete-registration lookup in crew work events aggregate

Risk Level: HIGH  
Status: ✅ Fixed  
File Path: src/repositories/event-crew.repository.ts  
Lines: 44-58, 248-252

Description:
**DRY / KISS.** Initial implementation always joined `eventathleteregistrations` and computed `athleteCount`, but the projection discarded it and the service used `countAthletesByEvents` for DTO mapping.

**Re-verification (`a535da7`):** Athlete-registration stages are split into `CREW_USER_WORK_EVENTS_ATHLETE_COUNT_STAGES` and appended only when `query.sortBy === "athletes"`, which is required for correct sort on the `athletes` column. Default list loads no longer pay for the extra join.

Impact:
- ~~Every crew-work events list request paid for a discarded join~~ Mitigated — join runs only when sorting by athletes.

Recommendation:
No further change required. Optional follow-up: document in a one-line comment on `CREW_USER_WORK_EVENTS_LIST_SORT_FIELD_MAP.athletes` that sort requires the conditional stages.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unused athlete-registration lookup in crew work events aggregate | HIGH | ✅ Fixed | src/repositories/event-crew.repository.ts | 44-58, 248-252 |

### Positive notes

- Conditional athlete lookup is the right trade-off: sort-by-athletes needs pipeline `athleteCount`; display counts stay on `countAthletesByEvents`.
- `seq` sort field added to `CREW_USER_WORK_EDIT_REQUEST_LIST_SORT_FIELD_MAP` for Request ID column sorting.
- Work list validation follows `createListQuerySchema` with entity-specific sort maps and edit-request status filter override.
- Routes place `/work` paths before `/:crewUserId` to avoid param shadowing.

**Merge readiness:** No open findings — ready to merge.
