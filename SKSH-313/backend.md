# Backend PR Review — skillshow (`SKSH-313`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-313`  
**Base:** `main...HEAD` (19 files, ~1413 insertions / ~169 deletions)  
**Scope:** Event view crew CRUD + server-driven sort/pagination for event athletes, crew, and mapped videos; `limit` ↔ `pageSize` alias in shared list validation (Critical & High only)  
**Findings:** 3 (1 Critical, 2 High)

---

---
Protected module changed: `list-query.validation.ts`

Risk Level: CRITICAL  
File Path: src/validation/list-query.validation.ts  
Lines: 37-78

Description:
**Protected module:** This PR modifies the frozen shared list-query schema: removes the Joi default on `pageSize`, adds a `limit` alias with custom coercion, and adds `.empty("")` on `status`. Every consumer of `createListQuerySchema` across the API now inherits this behavior change, not only event sub-tables.

Impact:
- Cross-cutting validation/pagination behavior changes outside the event ticket scope; regressions on partners, team users, crew users, etc. are possible if coercion or defaults diverge from prior Joi-only defaults.
- Violates the protected-module policy unless explicitly authorized as a platform-wide list-contract change.

Recommendation:
Either (a) revert the protected-file edit and handle `limit` → `pageSize` mapping only in event-specific query schemas/controllers until a dedicated list-contract ticket lands, or (b) if this alias is intentional platform-wide, document that SKSH-313 authorizes the protected-module change and add regression tests for at least one non-event list endpoint that uses `createListQuerySchema`. Do not expand protected-module edits further in this feature PR.

**PR comment (line 42):**  
**Critical (Protected module):** This PR changes frozen `list-query.validation.ts` (`limit` alias, `pageSize` default via custom, `status.empty("")`). Please revert and scope to event consumers, or split to an explicit list-contract ticket with cross-endpoint regression tests.

---

---
Crew name sort uses `firstName` only (athletes use computed full-name key)

Risk Level: HIGH  
File Path: src/constants/event.constants.ts  
Lines: 88-94  
File Path: src/repositories/event-crew.repository.ts  
Lines: 88-102

Description:
**Contract / correctness:** `EVENT_CREW_LIST_SORT_FIELD_MAP.crewName` maps to `userDoc.firstName`, while the athletes list builds `athleteNameSort` (display + first + last) before sorting. UI sends `sortBy=crewName` for the Crew Name column; two crew with the same first name sort arbitrarily by last name, and last-name-primary ordering is wrong.

Impact:
- Crew table sort does not match user expectation or athlete-tab behavior; admins see unstable or misleading name ordering.

Recommendation:
Mirror the athlete pipeline: add a `$addFields` `crewNameSort` (trimmed concat of first/last or display label) in `EVENT_CREW_LIST_PREPARE_STAGES`, map `crewName` → `crewNameSort` in `EVENT_CREW_LIST_SORT_FIELD_MAP`, and include `crewNameSort` in search fields if name search should stay aligned.

**PR comment (`event.constants.ts` line 90):**  
**High:** `crewName` sorts on `userDoc.firstName` only — athletes use a computed `athleteNameSort`. Please add a similar `crewNameSort` field so Crew Name column sorting matches the UI and athlete-tab behavior.

---

---
`listEventCrewAvailable` reads raw `req.query` fallback

Risk Level: HIGH  
File Path: src/controllers/event.controller.ts  
Lines: 236-237

Description:
**Contract:** The new `listEventCrewAvailable` handler casts `validatedQuery ?? req.query` even though the route is wired with `validate(eventCrewAvailableQuerySchema, "query")`. Main list handlers (`listEventCrew`, `listEventAthletes`, `listEventVideos`) correctly merge `DEFAULT_LIST_QUERY` with `validatedQuery` only.

Impact:
- Bypasses the established validated-query contract for a new endpoint; typed/coerced values may be skipped if middleware wiring changes, and the pattern diverges from the list endpoints refactored in the same PR.

Recommendation:
Read `req.validatedQuery` only (non-null assertion or guard), matching `listEventCrew`:

```typescript
const query = (req as ValidatedQueryRequest).validatedQuery as EventCrewAvailableQuery;
```

Apply the same fix to `listEventAthletesAvailable` if touching this area (pre-existing, same anti-pattern).

**PR comment (line 236):**  
**High (Contract):** `listEventCrewAvailable` falls back to raw `req.query` — please use `validatedQuery` only like the new `listEventCrew` / `listEventAthletes` handlers in this PR.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed: `list-query.validation.ts` | CRITICAL | Open | src/validation/list-query.validation.ts | 37-78 |
| 2 | Crew name sort uses `firstName` only | HIGH | Open | src/constants/event.constants.ts | 88-94 |
| 3 | `listEventCrewAvailable` reads raw `req.query` fallback | HIGH | Open | src/controllers/event.controller.ts | 236-237 |

**Positive notes:** Event crew follows established layering (controller → `event-crew.service` → `event-crew.repository`); list endpoints use `createListQuerySchema`, `runListQueryAggregate`, and `buildRegexOrMatch`; routes have auth + `validate()`; athlete/video lists migrated off in-memory filter/slice to aggregation; `event-crew.service.test.ts` and validation tests cover `limit` alias and video `sortBy` keys; frontend column keys (`crewName`, `athleteName`, `videoTitle`, etc.) align with backend `sortByValues`.

**Merge readiness:** **Not merge-ready** — 1 open Critical (protected module) and 2 open High findings.
