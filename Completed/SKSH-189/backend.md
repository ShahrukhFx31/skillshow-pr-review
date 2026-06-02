# Backend PR Review — skillshow (`SKSH-189`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-189`  
**Base:** `main...HEAD`  
**Re-verified:** `d854672` (`feat: add crew users superadmin flow`) — no backend commits after initial feature  
**Scope:** Crew users superadmin API — layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical & High only)  
**Findings:** 3 (0 Critical, 3 High accepted) — **0 open**

---

## Verification summary

| # | Title | Risk | Status | Evidence |
|---|--------|------|--------|----------|
| 1 | Reporting managers ignores `validatedQuery` | HIGH | **Accepted** | Team decision; not blocking SKSH-189 |
| 2 | Unbounded list aggregation | HIGH | **Accepted** | Team decision; unchanged in `crew-user.repository.ts` |
| 3 | Orphan user on failed crew insert | HIGH | **Accepted** | Team decision; unchanged in `crew-user.service.ts` |

**Regression re-check (PR diff):** **0 new** Critical or High issues in crew-user files since initial review.

---

---
Reporting managers endpoint ignores `validatedQuery`

**Status: Accepted (for now)** — team decision; not blocking SKSH-189.

Risk Level: HIGH  
File Path: src/controllers/crew-user.controller.ts  
Lines: 39-44

Description:
`GET /v1/crew-users/reporting-managers` is wired with `validate(reportingManagersQuerySchema, "query")`, but `listReportingManagers` still reads `req.query["search"]` directly instead of `req.validatedQuery` after Joi coercion/strip.

Impact:
- Coerced/trimmed query values from Joi are bypassed; behavior can diverge from other controllers that use `ValidatedQueryRequest`
- Inconsistent contract if the schema later adds transforms, defaults, or stricter types

Recommendation:
Read validated query values like other controllers:

```ts
import type { ValidatedQueryRequest } from "../types/common";

const query = (req as ValidatedQueryRequest).validatedQuery as { search?: string };
const search = typeof query.search === "string" ? query.search : undefined;
```

**PR comment (line 40):**  
**High:** This route validates query with Joi but reads raw `req.query.search`. Please use `req.validatedQuery` after `validate(..., "query")` so trimmed/coerced search matches the schema.

---

---
Unbounded crew user list with heavy aggregation

**Status: Accepted (for now)** — team decision; not blocking SKSH-189.

Risk Level: HIGH  
File Path: src/repositories/crew-user.repository.ts  
Lines: 95-127, 182-185

Description:
`listRows()` runs a fixed pipeline with three `$lookup` joins (user, creator, modifier), sort, and project, with no `limit`/`skip` or max cap. `GET /v1/crew-users` returns the full result set.

Impact:
- Response size and DB work grow linearly with every non-deleted crew user
- Admin list latency and memory use spike as the crew roster grows

Recommendation:
Add paginated list support (query params + `PAGINATION.MAX_LIMIT`), or cap with `limit` and align the admin UI to server-side paging.

---

---
`createCrewUser` can leave orphan `User` rows on failure

**Status: Accepted (for now)** — team decision; not blocking SKSH-189.

Risk Level: HIGH  
File Path: src/services/crew-user.service.ts  
Lines: 75-97

Description:
Creation calls `authRepository.createUser` and `user.save()`, then `crewUserRepository.insertCrewUser`. There is no transaction or compensating delete if `insertCrewUser` fails after the user document exists.

Impact:
- Orphan auth users without a `CrewUser` profile block re-create with the same email
- Partial failures require manual cleanup

Recommendation:
Wrap user + crew-user insert in a MongoDB transaction, or delete the user in `catch` if crew profile insert fails.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Reporting managers ignores `validatedQuery` | HIGH | Accepted | `src/controllers/crew-user.controller.ts` | 39-44 |
| 2 | Unbounded list aggregation | HIGH | Accepted | `src/repositories/crew-user.repository.ts` | 95-127, 182-185 |
| 3 | Orphan user on failed crew insert | HIGH | Accepted | `src/services/crew-user.service.ts` | 75-97 |

**Positive notes:** No backend changes since the feature commit; crew-user layer split, RBAC, Joi on body/params, and tests remain sound. Reporting-manager search in the repository still uses `containsInsensitive` with a limit of 50.

**Skipped (per severity / rules):** Sequential `bulkPatchCrewUsers` loop (mirrors skillshow-user). Welcome email non-blocking on create. No new files outside crew-user scope on this branch at `HEAD`.
