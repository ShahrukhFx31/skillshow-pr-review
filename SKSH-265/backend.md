# Backend PR Review — skillshow (`SKSH-265`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-01 @ `20bab61`  
**Scope:** Crew edit-request flow — feedback, assignment returns, crew dashboard, admin insights, internal revision, unread/read lanes (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 (0 Critical, 4 High) — **1 Open**, **3 Fixed**

### Protected modules

No changes in this PR to `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `audit-log.utils.ts`, or `change-stream.utils.ts`.

---

## GitHub comments (Open findings)

### 1. `src/controllers/edit-request.controller.ts` line 142

**High:** New list/dashboard handlers validate query with Joi but pass raw `req.query` into services — please read `(req as ValidatedQueryRequest).validatedQuery` for `getCrewDashboard`, `listAdminInternalRevisions`, `listAdminAssignmentReturns`, and `listAdminFeedbacks` so coerced `page`/`limit`/`module` match the validated contract (same pattern as `app-user.controller.ts` / `event.controller.ts`).

---

---
New list handlers bypass `validatedQuery`

Risk Level: HIGH  
File Path: src/controllers/edit-request.controller.ts  
Lines: 137-143, 355-381

Description:
**Contract.** Routes added/changed in this PR wire `validate(..., "query")` for crew dashboard, internal-revision list, and insights lists, but handlers pass `req.query` into services. Joi coerced values live on `req.validatedQuery` only (`validate.middleware.ts` documents that Express `req.query` is read-only and controllers must read list params from `validatedQuery`). Affected handlers: `getCrewDashboard`, `listAdminInternalRevisions`, `listAdminAssignmentReturns`, `listAdminFeedbacks`.

Impact:
- Typed/coerced list params may diverge from validated values; `stripUnknown` on validated query does not apply at the service boundary
- Inconsistent with established controllers (`app-user.controller.ts`, `event.controller.ts`, `crew-user.controller.ts`)

Recommendation:
```typescript
const query =
  (req as ValidatedQueryRequest).validatedQuery ?? req.query;

() => editRequestInsightsService.listAssignmentReturns(actorId, query),
```

Apply the same pattern in `getCrewDashboard` and `listAdminInternalRevisions`.

**PR comment (line 142):**  
**High:** New list/dashboard handlers validate query with Joi but pass raw `req.query` into services — please read `req.validatedQuery` so coerced `page`/`limit`/`module` match the validated contract.

**Status:** Open

---

---
Editor feedback endpoint allows duplicate submissions

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: 1870-1878

Description:
**Contract / data integrity.** `submitEditorFeedback` now checks `findLatestForEditRequestByRaisedBy` and throws `INVALID_TRANSITION:editor_feedback:already_submitted` before insert.

**Re-verification (@ `20bab61`):** Idempotency guard present at lines 1870–1878.

**Status:** ✅ Fixed

---

---
Read service bypasses repositories for insight read/unread

Risk Level: HIGH  
File Path: src/services/edit-request-read.service.ts  
Lines: 62-108

Description:
**Layer separation / DRY.** Unread summary and `markInsightRead` now delegate to repository helpers instead of direct Model calls.

**Re-verification (@ `20bab61`):** Uses `editRequestAssignmentReturnRepository` and `editRequestFeedbackRepository` for count/mark-read paths.

**Status:** ✅ Fixed

---

---
Admin-queue unread notify scales with admin user count

Risk Level: HIGH  
File Path: src/services/edit-request-read.service.ts  
Lines: 155-170

Description:
**Performance.** `notifyAdminQueueForEditRequest` now calls `bumpEditRequestUserReadBatch` with chunked `bulkWrite` via `editRequestUserReadRepository.bulkUpsertActivity`.

**Re-verification (@ `20bab61`):** Per-admin `Promise.all` removed.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | New list handlers bypass `validatedQuery` | HIGH | Open | src/controllers/edit-request.controller.ts | 137-143, 355-381 |
| 2 | Editor feedback endpoint allows duplicate submissions | HIGH | ✅ Fixed | src/services/edit-request.service.ts | 1870-1878 |
| 3 | Read service bypasses repositories for insight read/unread | HIGH | ✅ Fixed | src/services/edit-request-read.service.ts | 62-108 |
| 4 | Admin-queue unread notify scales with admin user count | HIGH | ✅ Fixed | src/services/edit-request-read.service.ts | 155-170 |

## Positive notes

- Layering for insights, crew dashboard, and read/unread is clear; routes use Joi + RBAC; crew return and internal-revision enforce assignment/manager checks.
- Batch `findLatestRatingByEditRequestIds` avoids N+1 on admin lists; `notifyEditRequestWorkflowEvent` centralizes notification fan-out.
- **DRY (good):** Shared read/unread utils; `buildEditRequestSourceVideoSummaries` embeds source video metadata on detail responses.
- **Protected modules:** None modified in this PR.

**Merge readiness:** **Not merge-ready** — 1 open High (`validatedQuery` on new list endpoints). Three prior High findings fixed on branch.
