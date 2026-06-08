# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-08 (full `/pr-review`) — @ `e7b62ea`  
**Archived:** 2026-06-08  
**Scope reviewed:** All **14 files** in `git diff main...HEAD` — app-user activity summary + paginated edit-request/video lists, coach teams server list + filter options, edit-request repo pagination, distribution-job distinct count, validation/routes/tests.  
**Prompts:** `backend-system-prompt.md`, `frontend-system-prompt.md` (DRY / KISS / Global consistency enforced)

**Aligned with:** [frontend.md](./frontend.md)

### Full review coverage

| File group | Reviewed |
|------------|----------|
| `app-user.*` (controller, service, routes, validation, constants, types, tests) | ✅ |
| `coach-link.repository.ts`, `coach-link.service.ts` | ✅ |
| `edit-request.repository.ts`, `edit-request.constants.ts`, `edit-request/index.ts` | ✅ |
| `distribution-job.repository.ts` | ✅ |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings this pass | 0 |
| Prior Open → Fixed (re-verified) | 1 |
| Prior Open → Accepted (re-verified) | 2 |
| Remaining Open | 0 |

---

## GitHub comments (Open findings)

None.

---

---
Activity edit-request rows return raw DB status

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 282-284

Description:
**Global consistency.** `mapActivityEditRequestRow` exposed `status: doc.status ?? ""` from the lean document. The established edit-request API contract uses `resolveEditRequestStatusForResponse` for source-review collapsing.

Impact:
- Activity edit-request table showed **wrong status** vs admin edit-request list for the same records.

Recommendation:
Resolve status in the mapper via `resolveEditRequestStatusForResponse(doc)`.

**Re-review (`817f68a`, confirmed @ `e7b62ea`):** ✅ **Fixed** — mapper calls `resolveEditRequestStatusForResponse` at lines 282-284.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Edit-request status filter matches DB field only

Risk Level: HIGH  
File Path: src/repositories/edit-request.repository.ts  
Lines: 161

Description:
**Global consistency.** When `opts.status` is set, `listByUserPaginated` applies an exact `$match` on the persisted `status` field. Validated values like `accepted` won't match documents that display as accepted but remain `submitted` in Mongo.

Impact:
- Server-side `status=accepted` filter may miss rows that resolve to accepted at response time.

Recommendation:
Expand Mongo filter to mirror `resolveEditRequestStatusForResponse`, or restrict validated filter values.

**PR comment (anchor `src/repositories/edit-request.repository.ts` line 161):** `listByUserPaginated` filters on raw Mongo `status` at line 161 (`filter["status"] = status`), but `accepted` is often only a resolved/display status while the DB stays `submitted`. Expand the filter query (or adjust validated values) so `status=accepted` matches what admins actually see.

**Accepted (2026-06-05):** Intentional for this slice — matches existing admin edit-request list (`buildListFilter` exact DB `status` match). Resolved/display status stays in the response layer; Mongo filter expansion deferred.

**Status:** Accepted

---

---
`in_progress` edit-request status added but never persisted

Risk Level: HIGH  
File Path: src/constants/edit-request.constants.ts  
Lines: 14

Description:
`IN_PROGRESS: "in_progress"` added to enum and validation; no workflow persists it yet.

Impact:
- Filter for `in_progress` returns empty until lifecycle is implemented.

Recommendation:
Remove from enum/validation or implement workflow.

**PR comment (anchor `src/constants/edit-request.constants.ts` line 14):** `in_progress` was added at line 14 (`IN_PROGRESS: "in_progress"`) to edit-request status constants and list validation, but nothing in the edit-request workflow ever sets that status. Remove it from the enum/validation or implement the lifecycle before exposing it as a filter value.

**Accepted (2026-06-05):** Intentional — reserved for upcoming lifecycle / frontend–backend alignment.

**Status:** Accepted

---

## Additional notes (not blockers)

- **Coach season sort** (`season` → `"year"` only): sorts by year, not full season label.
- **`findCoachTeamFilterSourceLean`**: unbounded `find({ coachUserId })` for filter options — acceptable for typical roster sizes.
- **Unused `status` query param** on coach-teams / activity-videos schemas (`APP_USER_STATUSES`): validated but ignored by services.
- **List contract:** New endpoints use `createListQuerySchema`, `validate(schema, "query")`, `validatedQuery` + `DEFAULT_LIST_QUERY` — aligned with frozen `list-query.validation.ts` (protected module unchanged ✅).

---

## Positive notes

- **Layering:** Controllers HTTP-only; no Mongoose in controller layer.
- **Activity split:** Summary counts decoupled from paginated list endpoints.
- **Coach teams:** Sport/season filter parsing, server sort, dedicated filter-options route (registered before `/:appUserId/teams`).
- **Counts:** `countDistinctVideoIdsByUserId` + `countByUserId` for summary tiles.
- **Tests:** Controller and service tests updated for paginated teams and summary-only activity.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Activity edit-request rows return raw DB status | HIGH | ✅ Fixed | `src/services/app-user.service.ts` | 282-284 |
| 2 | Edit-request status filter matches DB field only | HIGH | Accepted | `src/repositories/edit-request.repository.ts` | 161 |
| 3 | `in_progress` edit-request status added but never persisted | HIGH | Accepted | `src/constants/edit-request.constants.ts` | 14 |

**Merge readiness:** ✅ No open Critical/High blockers. All prior findings Fixed or Accepted; no new issues in `main...HEAD` at `e7b62ea`.
