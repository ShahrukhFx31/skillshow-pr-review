# Backend PR Review — skillshow (`SKSH-313`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-313`  
**Base:** `main...HEAD` @ `4293aec`  
**Re-verified:** 2026-06-08 (@ `4293aec`, merge-only since prior review — no new code findings)  
**Scope:** Event view crew CRUD + server-driven sort/pagination for event athletes, crew, and mapped videos; `getEventByRef` + `buildEventRefQueryFilter` (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 3 (0 Critical, 2 High) — **0 Open**, **3 Fixed**

### Protected modules

**Not modified:** `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`.

`limit` alias and empty-status handling live in event-scoped `createEventListQuerySchema` (`event.validation.ts`) only.

---

## GitHub comments (Open findings)

*None — all findings fixed on branch.*

---

---
Protected module changed: `list-query.validation.ts`

Risk Level: CRITICAL  
File Path: src/validation/event.validation.ts  
Lines: 36-68

Description:
Initial diff modified frozen shared list-query schema. **Re-verification:** `list-query.validation.ts` unchanged vs `main`; event routes use scoped `createEventListQuerySchema`.

**Status:** ✅ Fixed

---

---
Crew name sort uses `firstName` only

Risk Level: HIGH  
File Path: src/repositories/event-crew.repository.ts  
Lines: 53-70

Description:
`crewNameSort` aggregate field; `EVENT_CREW_LIST_SORT_FIELD_MAP.crewName` → `crewNameSort`.

**Status:** ✅ Fixed (@ `b63c391`)

---

---
`listEventCrewAvailable` reads raw `req.query` fallback

Risk Level: HIGH  
File Path: src/controllers/event.controller.ts  
Lines: 236-237

Description:
Handler uses `validatedQuery` only.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed: `list-query.validation.ts` | CRITICAL | ✅ Fixed | src/validation/event.validation.ts | 36-68 |
| 2 | Crew name sort uses `firstName` only | HIGH | ✅ Fixed | src/repositories/event-crew.repository.ts | 53-70 |
| 3 | `listEventCrewAvailable` reads raw `req.query` fallback | HIGH | ✅ Fixed | src/controllers/event.controller.ts | 236-237 |

### Re-review notes (2026-06-08 @ `4293aec`)

| Change | Verdict |
|--------|---------|
| `3380856` `getEventBySlug` → `getEventByRef` | `buildEventRefQueryFilter` consolidates slug/mongo/seq lookup; controller tests added |
| `findActiveByRef` → `findOneByRef` | DRY improvement in repository |
| Prior crew/list contract fixes | Unchanged on branch |

**Note:** Frontend `fetchAllEventRelatedList` for athletes/crew — **Accepted** per team (see [frontend.md](./frontend.md)).

**Positive notes:** Event crew layering; `createEventListQuerySchema` + `runListQueryAggregate`; auth + `validate()` on routes; crew service tests; swagger updated.

**Merge readiness:** **No open Critical/High blockers** — ticket review complete.
