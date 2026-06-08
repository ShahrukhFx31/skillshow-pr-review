# Backend PR Review — skillshow (`SKSH-265`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Scope:** Crew edit-request flow — editor feedback, assignment returns, crew dashboard, admin insights, internal revision, unread/read lanes (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract enforced)

**Findings:** 5 (0 Critical, 5 High) — **4 Open**, **1 Accepted**

> **Scope note:** Pagination / list-search performance findings are omitted per review request.

---

## GitHub comments (Open findings)

### 1. `src/services/edit-request.service.ts` line 1871

**High:** `submitEditorFeedback` always inserts a new row with no idempotency check — please reject or upsert when `findLatestForEditRequestByRaisedBy` already exists so repeat POSTs cannot skew crew KPI averages and insights.

### 2. `src/controllers/edit-request.controller.ts` lines 370, 380

**High:** Insights list routes validate query with Joi but pass raw `req.query` into services — please read `req.validatedQuery` (as in `app-user.controller.ts`) so coerced `page`/`limit`/`module` match the validated contract.

### 3. `src/services/edit-request-read.service.ts` line 64

**High:** Unread summary and `markInsightRead` call `EditRequestAssignmentReturnModel` / `EditRequestFeedbackModel` directly — please move count/update paths into the existing assignment-return and feedback repositories so Mongoose access stays in the data layer.

### 4. `src/services/edit-request-read.service.ts` line 174

**High:** `notifyAdminQueueForEditRequest` writes one `bumpEditRequestUserRead` per admin user on every queue event — please batch or fan-in (single aggregate unread lane) so high admin counts do not multiply DB writes per edit-request update.

---

---
Editor feedback endpoint allows duplicate submissions

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: 1851-1893

Description:
**Contract / data integrity.** `submitEditorFeedback` always inserts a new `EditRequestFeedback` row and never checks `findLatestForEditRequestByRaisedBy` (or a unique index on `module + editRequestId + raisedBy`). Repeat POSTs are accepted while status remains `completed`.

Impact:
- Duplicate ratings inflate crew dashboard `averageRating` and admin list `latestEditorRating`
- Insights Feedback tab shows multiple rows for one request; athlete UI can look submitted while API still accepts more ratings

Recommendation:
Reject a second submission for the same actor/request (`INVALID_TRANSITION` or 409), or upsert the existing row:

```typescript
const existing = await editRequestFeedbackRepository.findLatestForEditRequestByRaisedBy(
  editRequestId,
  actorId,
  EDIT_REQUEST_FEEDBACK_MODULE.EDIT_REQUEST,
);
if (existing) throw new Error("INVALID_TRANSITION:editor_feedback:already_submitted");
```

**PR comment (line 1871):**  
**High:** `submitEditorFeedback` always inserts a new row with no idempotency check — please reject or upsert when `findLatestForEditRequestByRaisedBy` already exists so repeat POSTs cannot skew crew KPI averages and insights.

**Status:** Open

---

---
New insights list handlers bypass `validatedQuery`

Risk Level: HIGH  
File Path: src/controllers/edit-request.controller.ts  
Lines: 365-381

Description:
**Contract.** `GET /admin/assignment-returns` and `GET /admin/feedbacks` wire `validate(adminEditRequestInsightsListQuerySchema, "query")`, but `listAdminAssignmentReturns` / `listAdminFeedbacks` pass `req.query` into `editRequestInsightsService`. Joi coerced values live on `req.validatedQuery` only (`validate.middleware.ts`); raw `req.query` is not replaced.

Impact:
- Typed/coerced list params may diverge from validated values; inconsistent with established controllers (`app-user.controller.ts`, `event.controller.ts`)
- Harder to rely on Joi defaults and `stripUnknown` at the service boundary

Recommendation:
```typescript
const query = (req as ValidatedQueryRequest).validatedQuery ?? req.query;
// ...
() => editRequestInsightsService.listAssignmentReturns(actorId, query),
```

Apply the same pattern to `getCrewDashboard` (line 142) and other query-validated routes touched in this PR when refactored.

**PR comment (line 370):**  
**High:** Insights list routes validate query with Joi but pass raw `req.query` into services — please read `req.validatedQuery` so coerced `page`/`limit`/`module` match the validated contract.

**Status:** Open

---

---
Read service bypasses repositories for insight read/unread

Risk Level: HIGH  
File Path: src/services/edit-request-read.service.ts  
Lines: 63-129

Description:
**Layer separation / DRY.** New `EditRequestReadService` calls `EditRequestAssignmentReturnModel` and `EditRequestFeedbackModel` directly for `countDocuments` (unread summary) and `updateOne` / `$push` (mark insight read). Repositories `edit-request-assignment-return.repository.ts` and `edit-request-feedback.repository.ts` already exist for these entities but lack read/unread helpers — logic is duplicated at the service layer instead.

Impact:
- Mongoose query shape for `readBy` arrays lives in the service; harder to reuse or test consistently with list pagination repos
- Future index/query changes require edits outside the repository layer

Recommendation:
Add `countUnreadForUser`, `markReadByUser` (or similar) to the assignment-return and feedback repositories; call those from `getUnreadSummary` and `markInsightRead`.

**PR comment (line 64):**  
**High:** Unread summary and `markInsightRead` call models directly — please move count/update paths into the existing repositories so Mongoose access stays in the data layer.

**Status:** Open

---

---
Admin-queue unread notify scales with admin user count

Risk Level: HIGH  
File Path: src/services/edit-request-read.service.ts  
Lines: 166-183

Description:
**Performance.** `notifyAdminQueueForEditRequest` loads every active user with edit-request read permission, then `Promise.all`s `bumpEditRequestUserRead` for each admin on every queue event (also invoked from `edit-request.utils.ts` notification paths).

Impact:
- Each edit-request lifecycle event triggers O(admins) DB writes; latency and load grow with admin roster size
- Bursty traffic (bulk status changes, socket-driven refreshes) amplifies write volume

Recommendation:
Batch upsert via repository helper (e.g. `bulkBumpUserReadForLane`), defer to a job queue, or maintain a single shared admin-queue counter keyed by `editRequestId` instead of per-admin documents when all admins share the same indicator semantics.

**PR comment (line 174):**  
**High:** `notifyAdminQueueForEditRequest` writes once per admin on every queue event — please batch or fan-in so admin count does not multiply DB writes per edit-request update.

**Status:** Open

---

---
Legacy editor rating requires one-time migration before release

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/scripts/migrate-edit-request-editor-feedback.ts  
Lines: 1-105

**Accepted:** Manual one-time migration is intentional; team will run `pnpm exec tsx src/scripts/migrate-edit-request-editor-feedback.ts` in staging/prod as part of release (not wired into deploy startup).

Description:
Feedback now lives in `EditRequestFeedback`; participant responses expose `latestFeedback` / `myLatestFeedback` from that collection. The migration script copies legacy `editorRating` / `editorFeedback` / `editorFeedbackAt` from existing MongoDB documents and unsets those fields.

Impact:
- Production requests with legacy fields only will show no athlete/editor rating after deploy until migration runs

Recommendation:
Run the script in each environment as part of release checklist; verify migrated vs skipped counts in staging.

**PR comment (line 5):**  
Accepted — manual migration at release is acknowledged.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Editor feedback endpoint allows duplicate submissions | HIGH | Open | src/services/edit-request.service.ts | 1851-1893 |
| 2 | New insights list handlers bypass `validatedQuery` | HIGH | Open | src/controllers/edit-request.controller.ts | 365-381 |
| 3 | Read service bypasses repositories for insight read/unread | HIGH | Open | src/services/edit-request-read.service.ts | 63-129 |
| 4 | Admin-queue unread notify scales with admin user count | HIGH | Open | src/services/edit-request-read.service.ts | 166-183 |
| 5 | Legacy editor rating requires one-time migration before release | HIGH | Accepted | src/scripts/migrate-edit-request-editor-feedback.ts | 1-105 |

## Positive notes

- Layering for insights, crew dashboard, and read/unread is clear; routes use Joi + RBAC; crew return and internal-revision paths enforce assignment/manager checks.
- Batch `findLatestRatingByEditRequestIds` avoids N+1 on admin lists; unread summary uses dedicated read service + repository.
- **Protected modules:** No changes to `list-query.validation.ts`, `list-query-aggregation.utils.ts`, or `list-row-repository.utils.ts`.
- **DRY (good):** Centralized notification workflow via `notifyEditRequestWorkflowEvent`; shared read/unread utils (`edit-request-user-read.utils.ts`).

**Merge readiness:** **Not merge-ready** — 4 open High blockers (duplicate editor feedback, `validatedQuery`, read-service layer bypass, admin-queue N× writes). Migration item Accepted.
