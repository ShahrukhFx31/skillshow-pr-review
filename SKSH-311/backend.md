# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-05 (full `/pr-review`) — @ `817f68a`  
**Scope reviewed:** All **14 files** in `git diff main...HEAD` — app-user activity summary + paginated edit-request/video lists, coach teams server list + filter options, edit-request repo pagination, distribution-job distinct count, validation/routes/tests.  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency enforced)

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
| Prior Open → Fixed this pass | 1 |
| Prior Open → Accepted this pass | 2 |
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

**Re-review (`817f68a`):** ✅ **Fixed** — mapper now calls `resolveEditRequestStatusForResponse` at lines 282-284.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Edit-request status filter matches DB field only

Risk Level: HIGH  
File Path: src/repositories/edit-request.repository.ts  
Lines: 161

Description:
**Global consistency.** When `opts.status` is set, `listByUserPaginated` applies an exact `$match` on the persisted `status` field. The validation schema accepts `accepted`, but many “accepted for editing” requests remain `submitted` in Mongo (per `EDIT_REQUEST_STATUS.ACCEPTED` comment) and only **resolve** to `accepted` at response time. Filtering `status=accepted` therefore misses most accepted rows; `status=submitted` mixes pending-review and accepted-ready rows.

Impact:
- Server-side status filter on the activity endpoint is **semantically broken** for `accepted` / pending-vs-accepted distinctions, even after frontend maps UI keys (`pending_review` → `submitted`, `accepted` → `accepted`).
- Admin users cannot reliably filter activity edit requests by the statuses they see in the table.

Recommendation:
Either (a) expand the repository filter for `accepted` to include source-review-phase `submitted` docs where all `videoReviews` are accepted and `editorId` is null (mirror `resolveEditRequestStatusForResponse` logic in query form), or (b) document that only raw DB statuses are filterable and remove `accepted` from validated filter values until query support exists. Prefer (a) for parity with admin list UX.

**PR comment (anchor `src/repositories/edit-request.repository.ts` line 161):** `listByUserPaginated` filters on raw Mongo `status` at line 161 (`filter["status"] = status`), but `accepted` is often only a resolved/display status while the DB stays `submitted`. Expand the filter query (or adjust validated values) so `status=accepted` matches what admins actually see.

**Accepted (2026-06-05):** Intentional for this slice — matches existing admin edit-request list (`buildListFilter` exact DB `status` match). Resolved/display status stays in the response layer (`resolveEditRequestStatusForResponse`); expanding the Mongo filter is deferred follow-up, not a merge blocker for SKSH-311.

**Status:** Accepted

---

---
`in_progress` edit-request status added but never persisted

Risk Level: HIGH  
File Path: src/constants/edit-request.constants.ts  
Lines: 14

Description:
`IN_PROGRESS: "in_progress"` was added to `EDIT_REQUEST_STATUS`, `EditRequestStatus`, the Mongoose schema enum, and `EDIT_REQUEST_STATUS_VALUES` (used by the new activity edit-request list validation). No service, state machine (`ACTION_GUARD`), or transition in this repo writes `in_progress` to edit-request documents.

Impact:
- API and Joi validation advertise a status that **does not exist** in production data.
- Any client filter for `in_progress` always returns an empty page.
- Schema enum drift without a defined lifecycle.

Recommendation:
Remove `IN_PROGRESS` from edit-request constants/types/validation until a workflow actually persists it, or implement the transition that sets `status: "in_progress"` and add tests. Do not expose it in `EDIT_REQUEST_STATUS_VALUES` prematurely.

**PR comment (anchor `src/constants/edit-request.constants.ts` line 14):** `in_progress` was added at line 14 (`IN_PROGRESS: "in_progress"`) to edit-request status constants and list validation, but nothing in the edit-request workflow ever sets that status. Remove it from the enum/validation or implement the lifecycle before exposing it as a filter value.

**Accepted (2026-06-05):** Intentional — status reserved for upcoming lifecycle / frontend–backend alignment (labels already added on admin UI). No documents use it yet; empty filter results are acceptable until workflow lands in a follow-up ticket.

**Status:** Accepted

---

## Additional notes (not blockers)

- **Coach season sort** (`APP_USER_COACH_TEAMS_LIST_SORT_FIELD_MAP.season` → `"year"` only): sorting by Season orders by year, ignoring the `season` label.
- **`findCoachTeamFilterSourceLean`**: unbounded `find({ coachUserId })` for filter options — acceptable for typical roster sizes.
- **Unused `status` query param** on coach-teams / activity-videos schemas (`APP_USER_STATUSES`): validated but ignored by services.

---

## Positive notes

- **Finding 1 fixed:** `mapActivityEditRequestRow` now uses `resolveEditRequestStatusForResponse` — aligned with admin edit-request list contract.
- **Layering:** Controllers HTTP-only; `validatedQuery` + `DEFAULT_LIST_QUERY` on new list endpoints.
- **Activity split:** Summary counts decoupled from paginated list endpoints.
- **Coach teams:** Sport/season filter parsing, server sort, filter-options endpoint.
- **Tests:** Controller and service tests updated for paginated teams and summary-only activity.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Activity edit-request rows return raw DB status | HIGH | ✅ Fixed | `src/services/app-user.service.ts` | 282-284 |
| 2 | Edit-request status filter matches DB field only | HIGH | Accepted | `src/repositories/edit-request.repository.ts` | 161 |
| 3 | `in_progress` edit-request status added but never persisted | HIGH | Accepted | `src/constants/edit-request.constants.ts` | 14 |

**Merge readiness:** ✅ No open Critical/High blockers. Findings #2–#3 accepted per team — raw DB status filter matches admin list pattern; `in_progress` reserved for follow-up lifecycle.
