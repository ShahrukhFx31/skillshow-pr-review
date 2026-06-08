# Backend PR Review — skillshow (`SKSH-265`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-08 @ `e5b1164`  
**Scope:** Crew edit-request flow — feedback, assignment returns, crew dashboard, admin insights, internal revision, unread/read lanes (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 5 (0 Critical, 5 High) — **4 Open**, **1 Accepted**

> **Scope note:** Pagination / list-search performance findings (insights search `$in` resolution, unbounded export fetches, etc.) are omitted per review request.

### Protected modules

No changes in this PR to `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `audit-log.utils.ts`, or `change-stream.utils.ts`.

---

## GitHub comments (Open findings)

### 1. `src/services/edit-request.service.ts` line 1871

**High:** `submitEditorFeedback` always inserts a new row with no idempotency check — please reject or upsert when `findLatestForEditRequestByRaisedBy` already exists so repeat POSTs cannot skew crew KPI averages and insights.

### 2. `src/controllers/edit-request.controller.ts` line 370

**High:** Insights list routes validate query with Joi but pass raw `req.query` into services — please read `req.validatedQuery` so coerced `page`/`limit`/`module` match the validated contract (same for `getCrewDashboard` at line 142).

### 3. `src/services/edit-request-read.service.ts` line 64

**High:** Unread summary and `markInsightRead` call Mongoose models directly — please move count/update paths into the assignment-return and feedback repositories.

### 4. `src/services/edit-request-read.service.ts` line 174

**High:** `notifyAdminQueueForEditRequest` writes one `bumpEditRequestUserRead` per admin on every queue event — please batch or fan-in so admin count does not multiply DB writes per edit-request update.

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
- Insights Feedback tab shows multiple rows for one request

Recommendation:
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
New list handlers bypass `validatedQuery`

Risk Level: HIGH  
File Path: src/controllers/edit-request.controller.ts  
Lines: 137-143, 365-381

Description:
**Contract.** Routes wire `validate(..., "query")` for crew dashboard and insights lists, but handlers pass `req.query` into services. Joi coerced values live on `req.validatedQuery` only (`validate.middleware.ts`); raw `req.query` is not replaced. New endpoints in this PR (`getCrewDashboard`, `listAdminAssignmentReturns`, `listAdminFeedbacks`) follow the same anti-pattern.

Impact:
- Typed/coerced list params may diverge from validated values; inconsistent with `app-user.controller.ts` / `event.controller.ts`
- `stripUnknown` on validated query does not apply at the service boundary

Recommendation:
```typescript
const query = (req as ValidatedQueryRequest).validatedQuery ?? req.query;
() => editRequestInsightsService.listAssignmentReturns(actorId, query),
```

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
**Layer separation / DRY.** `EditRequestReadService` calls `EditRequestAssignmentReturnModel` and `EditRequestFeedbackModel` directly for `countDocuments` (unread summary) and `updateOne` / `$push` (`markInsightRead`). Repositories for these entities exist but lack read/unread helpers — Mongoose access is duplicated in the service layer.

Impact:
- `readBy` query logic is harder to reuse with list pagination repos
- Index/query changes require service edits instead of repository-only updates

Recommendation:
Add `countUnreadForUser` / `markReadByUser` to `edit-request-assignment-return.repository.ts` and `edit-request-feedback.repository.ts`; call from `getUnreadSummary` and `markInsightRead`.

**PR comment (line 64):**  
**High:** Unread summary and `markInsightRead` call models directly — please move count/update paths into the existing repositories.

**Status:** Open

---

---
Admin-queue unread notify scales with admin user count

Risk Level: HIGH  
File Path: src/services/edit-request-read.service.ts  
Lines: 166-183

Description:
**Performance.** `notifyAdminQueueForEditRequest` loads every active user with edit-request read permission, then `Promise.all`s `bumpEditRequestUserRead` per admin. Invoked from notification paths in `edit-request.utils.ts` on lifecycle events.

Impact:
- O(admins) DB writes per edit-request event; latency grows with admin roster
- Bursty updates amplify write load

Recommendation:
Batch upsert via a repository helper, defer to a job, or use a shared admin-queue counter when all admins share the same indicator semantics.

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

**Accepted:** Manual one-time migration is intentional; team will run `pnpm exec tsx src/scripts/migrate-edit-request-editor-feedback.ts` in staging/prod as part of release.

**PR comment (line 5):** Accepted — manual migration at release is acknowledged.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Editor feedback endpoint allows duplicate submissions | HIGH | Open | src/services/edit-request.service.ts | 1851-1893 |
| 2 | New list handlers bypass `validatedQuery` | HIGH | Open | src/controllers/edit-request.controller.ts | 137-143, 365-381 |
| 3 | Read service bypasses repositories for insight read/unread | HIGH | Open | src/services/edit-request-read.service.ts | 63-129 |
| 4 | Admin-queue unread notify scales with admin user count | HIGH | Open | src/services/edit-request-read.service.ts | 166-183 |
| 5 | Legacy editor rating requires one-time migration before release | HIGH | Accepted | src/scripts/migrate-edit-request-editor-feedback.ts | 1-105 |

## Positive notes

- Layering for insights, crew dashboard, and read/unread is clear; routes use Joi + RBAC; crew return and internal-revision enforce assignment/manager checks.
- Batch `findLatestRatingByEditRequestIds` avoids N+1 on admin lists; `notifyEditRequestWorkflowEvent` centralizes notification fan-out.
- **DRY (good):** Shared read/unread utils (`edit-request-user-read.utils.ts`); internal-revision review now requires explicit `outputId`/`versionId` in validation.
- **Protected modules:** None modified in this PR.

**Merge readiness:** **Not merge-ready** — 4 open High blockers. Migration Accepted.
