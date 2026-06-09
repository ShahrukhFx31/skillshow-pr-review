# Backend PR Review — skillshow (`SKSH-265`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-01 @ `d15f948`  
**Scope:** Crew edit-request flow — feedback, assignment returns, crew dashboard, admin insights, internal revision, unread/read lanes (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 (0 Critical, 4 High) — **0 Open**, **3 Fixed**, **1 Accepted**

### Protected modules

No changes in this PR to `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `audit-log.utils.ts`, or `change-stream.utils.ts`.

---

## GitHub comments (Open findings)

_None — all findings resolved or accepted._

---

---
New list handlers bypass `validatedQuery`

Risk Level: HIGH  
File Path: src/controllers/edit-request.controller.ts  
Lines: 137-143, 355-381

Description:
**Contract.** Routes added in this PR wire `validate(..., "query")` for crew dashboard, internal-revision list, and insights lists, but handlers pass `req.query` into services instead of `req.validatedQuery`.

**Accepted:** Acknowledged. Edit-request list handlers (including pre-existing `list`, `listAdmin`, `listAdminAssigned`) follow the same `req.query` + service-side `parsePagination` pattern. Joi still gates invalid input at the route; services coerce `page`/`limit` with shared bounds. Migrating the whole edit-request controller to `validatedQuery` is deferred to a follow-up scoped to list-contract alignment — not a blocker for this feature PR.

**PR comment (line 142):** Accepted — deferred; consistent with existing edit-request list handlers in this controller.

**Status:** Accepted

---

---
Editor feedback endpoint allows duplicate submissions

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: 1878-1886

Description:
**Contract / data integrity.** `submitEditorFeedback` checks `findLatestForEditRequestByRaisedBy` and throws `INVALID_TRANSITION:editor_feedback:already_submitted` before insert. Controller maps this to a client-safe 409 response.

**Re-verification (@ `d15f948`):** Idempotency guard present.

**Status:** ✅ Fixed

---

---
Read service bypasses repositories for insight read/unread

Risk Level: HIGH  
File Path: src/services/edit-request-read.service.ts  
Lines: 62-108

Description:
**Layer separation / DRY.** Unread summary and `markInsightRead` delegate to `editRequestAssignmentReturnRepository` and `editRequestFeedbackRepository` helpers.

**Re-verification (@ `d15f948`):** No direct Model calls in read service.

**Status:** ✅ Fixed

---

---
Admin-queue unread notify scales with admin user count

Risk Level: HIGH  
File Path: src/services/edit-request-read.service.ts  
Lines: 155-170

Description:
**Performance.** `notifyAdminQueueForEditRequest` calls `bumpEditRequestUserReadBatch` with chunked `bulkWrite` via `editRequestUserReadRepository.bulkUpsertActivity`.

**Re-verification (@ `d15f948`):** Per-admin `Promise.all` removed.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | New list handlers bypass `validatedQuery` | HIGH | Accepted | src/controllers/edit-request.controller.ts | 137-143, 355-381 |
| 2 | Editor feedback endpoint allows duplicate submissions | HIGH | ✅ Fixed | src/services/edit-request.service.ts | 1878-1886 |
| 3 | Read service bypasses repositories for insight read/unread | HIGH | ✅ Fixed | src/services/edit-request-read.service.ts | 62-108 |
| 4 | Admin-queue unread notify scales with admin user count | HIGH | ✅ Fixed | src/services/edit-request-read.service.ts | 155-170 |

## Positive notes

- Layering for insights, crew dashboard, and read/unread is clear; routes use Joi + RBAC; crew return and internal-revision enforce assignment/manager checks.
- Batch `findLatestRatingByEditRequestIds` avoids N+1 on admin lists; `notifyEditRequestWorkflowEvent` centralizes notification fan-out.
- **DRY (good):** Shared read/unread utils; `buildEditRequestSourceVideoSummaries` embeds source video metadata on detail responses.
- **Protected modules:** None modified in this PR.

**Merge readiness:** **Merge-ready** — no open blockers (1 Accepted, 3 Fixed).
