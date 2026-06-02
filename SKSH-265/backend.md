# Backend PR Review — skillshow (`SKSH-265`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-265`  
**Base:** `main...HEAD`  
**Scope:** Crew edit-request flow — editor feedback, assignment returns, crew dashboard, admin insights, internal revision (Critical & High only)  
**Findings:** 2 (0 Critical, 2 High) — **1 Open**, **1 Accepted**

> **Scope note:** Pagination / list-search performance findings (insights search `$in` resolution, page/limit caps, etc.) are omitted per review request.

---

---
Editor feedback endpoint allows duplicate submissions

Risk Level: HIGH
File Path: src/services/edit-request.service.ts
Lines: 1510-1552

Description:
`submitEditorFeedback` always inserts a new `EditRequestFeedback` row and never checks `findLatestForEditRequestByRaisedBy` (or a unique index on `module + editRequestId + raisedBy`). Repeat POSTs are accepted while status remains `completed`.

Impact:
- Duplicate ratings inflate crew dashboard `averageRating` and admin list `latestEditorRating`
- Insights Feedback tab shows multiple rows for one request; athlete UI can look submitted while API still accepts more ratings

Recommendation:
Reject a second submission for the same actor/request (`INVALID_TRANSITION` or 409), or upsert the existing row. Add a partial unique index if one rating per athlete per request is the rule:

```typescript
const existing = await editRequestFeedbackRepository.findLatestForEditRequestByRaisedBy(
  editRequestId,
  actorId,
  EDIT_REQUEST_FEEDBACK_MODULE.EDIT_REQUEST,
);
if (existing) throw new Error("INVALID_TRANSITION:editor_feedback:already_submitted");
```

**PR comment (line 1530):**  
**High:** Please enforce one editor-feedback submission per athlete per edit request (check existing row or unique index) so repeat POSTs cannot skew crew KPI averages and insights.

---

---
Legacy editor rating requires one-time migration before release

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/scripts/migrate-edit-request-editor-feedback.ts  
Lines: 1-105

**Accepted:** Manual one-time migration is intentional; team will run `pnpm exec tsx src/scripts/migrate-edit-request-editor-feedback.ts` in staging/prod as part of release (not wired into deploy startup).

Description:
Feedback now lives in `EditRequestFeedback`; participant responses expose `latestFeedback` / `myLatestFeedback` from that collection. The migration script copies legacy `editorRating` / `editorFeedback` / `editorFeedbackAt` from existing MongoDB documents and unsets those fields. The script is manual (`pnpm exec tsx ...`) and is not wired into deploy startup.

Impact:
- Production requests with legacy fields only will show no athlete/editor rating after deploy until migration runs
- Crew average-rating KPI and admin feedback insights will be wrong or empty for historical work

Recommendation:
Run `migrate-edit-request-editor-feedback.ts` against each environment before or immediately after deploy, and document it in the release checklist. Verify counts migrated vs skipped in staging.

**PR comment (line 5):**  
**High:** Please confirm this migration runs in staging/prod as part of release (`pnpm exec tsx src/scripts/migrate-edit-request-editor-feedback.ts`) — without it, historical editor ratings on old edit requests will not appear in the new feedback APIs or crew KPI.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Editor feedback endpoint allows duplicate submissions | HIGH | Open | src/services/edit-request.service.ts | 1510-1552 |
| 2 | Legacy editor rating requires one-time migration before release | HIGH | Accepted | src/scripts/migrate-edit-request-editor-feedback.ts | 1-105 |

**Positive notes:** Layering for insights (service + repositories + access util) and crew dashboard is clear; routes use Joi + RBAC; crew return and internal-revision paths enforce assignment/manager checks; batch `findLatestRatingByEditRequestIds` avoids N+1 on admin lists.
