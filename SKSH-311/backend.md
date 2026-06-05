# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (`/pr-review` full pass) — @ `origin/SKSH-311` → `0b5383b`  
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
| Prior Open → Fixed this pass | 0 |
| Remaining Open | 3 |

---

## GitHub comments (Open findings)

### Finding 1 — `src/services/app-user.service.ts` line 280

> Activity edit-request rows return raw Mongo `status` at line 280 (`status: doc.status ?? ""`), but admin lists use `resolveEditRequestStatusForResponse` for source-review collapsing. Apply the same resolver in `mapActivityEditRequestRow` so statuses match the rest of the edit-request API.

`mapActivityEditRequestRow` returns raw `doc.status`, but every other admin edit-request list/detail path uses `resolveEditRequestStatusForResponse(doc)` so source-review phase collapses `submitted` → `accepted` / `editor_assigned` correctly. Activity rows will show stale DB statuses (e.g. `submitted` when admin list would show `accepted`).

### Finding 2 — `src/repositories/edit-request.repository.ts` line 161

> `listByUserPaginated` filters on raw Mongo `status` at line 161 (`filter["status"] = status`), but `accepted` is often only a resolved/display status while the DB stays `submitted`. Expand the filter query (or adjust validated values) so `status=accepted` matches what admins actually see.

`listByUserPaginated` filters with `filter["status"] = status` (exact DB match). Validated values like `accepted` won't match documents that **display** as accepted but remain `submitted` in Mongo until an editor is assigned — same semantic gap as the response-mapping issue. Status filtering on this new endpoint will return wrong or empty pages even after the frontend maps UI keys.

### Finding 3 — `src/constants/edit-request.constants.ts` line 14

> `in_progress` was added at line 14 (`IN_PROGRESS: "in_progress"`) to edit-request status constants and list validation, but nothing in the edit-request workflow ever sets that status. Remove it from the enum/validation or implement the lifecycle before exposing it as a filter value.

`IN_PROGRESS: "in_progress"` was added to `EDIT_REQUEST_STATUS`, the Mongoose enum, and `EDIT_REQUEST_STATUS_VALUES`, but no edit-request workflow sets this status. It is now a valid filter value that can never match real rows.

---

---
Activity edit-request rows return raw DB status

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 280

Description:
**Global consistency.** `mapActivityEditRequestRow` exposes `status: doc.status ?? ""` from the lean document. The established edit-request API contract uses `resolveEditRequestStatusForResponse` (see `edit-request.service.ts` `toAdminListItem`, `enrichEditRequestForResponse`) to derive client-visible status from source-video reviews (`submitted` + all sources accepted → `accepted`, with editor → `editor_assigned`, etc.).

Impact:
- Activity edit-request table shows **wrong status** vs admin edit-request list for the same records.
- Frontend `mapBackendStatusToUiStatus` cannot fully compensate without `videoReviews` context on each row.

Recommendation:
Resolve status in the mapper:

```typescript
import { resolveEditRequestStatusForResponse } from "../utils/edit-request-source-review.utils";

private mapActivityEditRequestRow(doc: ...): AppUserActivityEditRequestRow {
  return {
    id: String(doc._id),
    title: doc.editPreferences?.title ?? doc.name ?? "—",
    submittedDate: doc.createdAt instanceof Date ? doc.createdAt.toISOString() : "",
    status: resolveEditRequestStatusForResponse(doc),
    assignedCrew: ...
  };
}
```

**PR comment (anchor `src/services/app-user.service.ts` line 280):** Activity edit-request rows return raw Mongo `status` at line 280 (`status: doc.status ?? ""`), but admin lists use `resolveEditRequestStatusForResponse` for source-review collapsing. Apply the same resolver in `mapActivityEditRequestRow` so statuses match the rest of the edit-request API.

**Status:** Open

---

---
Edit-request status filter matches DB field only

Risk Level: HIGH  
File Path: src/repositories/edit-request.repository.ts  
Lines: 161

Description:
**Global consistency.** When `opts.status` is set, `listByUserPaginated` applies an exact `$match` on the persisted `status` field. The validation schema accepts `accepted`, but many “accepted for editing” requests remain `submitted` in Mongo (per `EDIT_REQUEST_STATUS.ACCEPTED` comment) and only **resolve** to `accepted` at response time. Filtering `status=accepted` therefore misses most accepted rows; `status=submitted` mixes pending-review and accepted-ready rows.

Impact:
- Server-side status filter on the new activity endpoint is **semantically broken** for `accepted` / pending-vs-accepted distinctions, even if the frontend maps UI keys correctly.
- Admin users cannot reliably filter activity edit requests by the statuses they see in the table.

Recommendation:
Either (a) expand the repository filter for `accepted` to include source-review-phase `submitted` docs where all `videoReviews` are accepted and `editorId` is null (mirror `resolveEditRequestStatusForResponse` logic in query form), or (b) document that only raw DB statuses are filterable and remove `accepted` from validated filter values until query support exists. Prefer (a) for parity with admin list UX.

**PR comment (anchor `src/repositories/edit-request.repository.ts` line 161):** `listByUserPaginated` filters on raw Mongo `status` at line 161 (`filter["status"] = status`), but `accepted` is often only a resolved/display status while the DB stays `submitted`. Expand the filter query (or adjust validated values) so `status=accepted` matches what admins actually see.

**Status:** Open

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

**Status:** Open

---

## Additional notes (not blockers)

- **Coach season sort** (`APP_USER_COACH_TEAMS_LIST_SORT_FIELD_MAP.season` → `"year"` only): sorting by Season orders by year, ignoring the `season` label — may look wrong when multiple seasons share a year.
- **`findCoachTeamFilterSourceLean`**: unbounded `find({ coachUserId })` for filter options — acceptable for typical roster sizes; watch for coaches with very large team counts.
- **Unused `status` query param** on coach-teams / activity-videos schemas (`APP_USER_STATUSES`): validated but ignored by services — minor API-surface noise.

---

## Positive notes

- **Layering:** Controllers stay HTTP-only; new list endpoints use `validatedQuery` spread with `DEFAULT_LIST_QUERY`.
- **Activity split:** Summary counts decoupled from paginated list endpoints — correct shape for server tables.
- **Coach teams:** Sport/season filter parsing, server sort, and filter-options endpoint follow established list-query patterns.
- **Counts:** `countDistinctVideoIdsByUserId` + `countByUserId` are reasonable summary abstractions.
- **Tests:** Controller and service tests updated for paginated teams and summary-only activity.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Activity edit-request rows return raw DB status | HIGH | Open | `src/services/app-user.service.ts` | 280 |
| 2 | Edit-request status filter matches DB field only | HIGH | Open | `src/repositories/edit-request.repository.ts` | 161 |
| 3 | `in_progress` edit-request status added but never persisted | HIGH | Open | `src/constants/edit-request.constants.ts` | 14 |

**Merge readiness:** ❌ Blocked — 3 open **High** backend findings on edit-request status semantics. Frontend status-filter issues (#1–2 in `frontend.md`) compound the same area; fix backend response/filter contract and frontend mapping together.
