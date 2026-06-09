# Backend PR Review — skillshow (`SKSH-313`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-313`  
**Base:** `main...HEAD` @ `b63c391`  
**Re-verified:** 2026-06-08 (@ `b63c391`, unchanged since prior review)  
**Scope:** Event view crew CRUD + server-driven sort/pagination for event athletes, crew, and mapped videos; event-scoped `limit` ↔ `pageSize` alias (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 3 (0 Critical, 2 High) — **0 Open**, **3 Fixed**

### Protected modules

**Not modified:** `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`.

`limit` alias and empty-status handling live in event-scoped `createEventListQuerySchema` (`event.validation.ts`) only.

---

## GitHub comments (Open findings)

*None — all prior findings fixed on branch.*

---

---
Protected module changed: `list-query.validation.ts`

Risk Level: CRITICAL  
File Path: src/validation/list-query.validation.ts  
Lines: 37-78 (initial review)

Description:
Initial diff modified the frozen shared list-query schema. **Re-verification (@ `b63c391`):** `list-query.validation.ts` is **unchanged vs `main`**. Event sub-table routes use `createEventListQuerySchema` in `event.validation.ts`, which wraps `createListQuerySchema` and adds `limit` → `pageSize` coercion locally.

**Status:** ✅ Fixed

---

---
Crew name sort uses `firstName` only (athletes use computed full-name key)

Risk Level: HIGH  
File Path: src/constants/event.constants.ts  
Lines: 89-90  
File Path: src/repositories/event-crew.repository.ts  
Lines: 53-70

Description:
**Contract / correctness.** `EVENT_CREW_LIST_SORT_FIELD_MAP.crewName` now maps to `crewNameSort`; repository `$addFields` builds trimmed first/last concat and search includes `crewNameSort`.

**Re-verification (@ `b63c391`, commit `b63c391`):** Aligns with athlete `athleteNameSort` pattern.

**Status:** ✅ Fixed

---

---
`listEventCrewAvailable` reads raw `req.query` fallback

Risk Level: HIGH  
File Path: src/controllers/event.controller.ts  
Lines: 236-237

Description:
**Contract.** Handler now reads `(req as ValidatedQueryRequest).validatedQuery as EventCrewAvailableQuery` only (no `req.query` fallback). `listEventAthletesAvailable` updated the same way.

**Re-verification (@ `b63c391`):** Validated-query contract satisfied.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed: `list-query.validation.ts` | CRITICAL | ✅ Fixed | src/validation/event.validation.ts | 36-68 |
| 2 | Crew name sort uses `firstName` only | HIGH | ✅ Fixed | src/repositories/event-crew.repository.ts | 53-70 |
| 3 | `listEventCrewAvailable` reads raw `req.query` fallback | HIGH | ✅ Fixed | src/controllers/event.controller.ts | 236-237 |

### Re-review notes (2026-06-08)

| Change | Verdict |
|--------|---------|
| `b63c391` `crewNameSort` aggregate field | Fixes crew name sort vs athlete tab |
| `limit` alias scoped to `createEventListQuerySchema` | Protected `list-query.validation.ts` untouched |
| `validatedQuery` on crew/athlete available handlers | Contract aligned with main list endpoints |

**Positive notes:** Event crew follows controller → service → repository layering; list endpoints use `createEventListQuerySchema`, `runListQueryAggregate`, and `buildRegexOrMatch`; routes have auth + `validate()`; athlete/video lists migrated off in-memory filter/slice; tests cover crew service and `limit` alias.

**Merge readiness:** **No open Critical/High blockers on the backend diff.**
