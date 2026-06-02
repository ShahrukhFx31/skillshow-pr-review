# Backend PR Review — skillshow (`SKSH-305`)

**Repo:** skillshow  
**Branch:** `SKSH-305`  
**Base:** `main...HEAD`  
**Scope:** Layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical and High only)  
**Findings:** 1 (0 Critical, 1 High) — **re-verified (task complete): ✅ Fixed**

---

---
Deleting a library video leaves orphaned active edit requests

Risk Level: HIGH
File Path: src/services/edit-request.service.ts
Lines: 108-141
Primary PR Comment Line: 130

Description:
When a user deletes a source video from the library, `detachVideoFromUserEditRequests()` removes the video reference but does not transition lifecycle state when the request is left with zero source videos. This creates requests that remain in actionable statuses (`change_requested`, `change_request_submitted`, etc.) even though they no longer have any source media to resolve.

Impact:
- Admin and user lists can keep showing an edit request as active/workable while it is no longer fulfillable.
- Lifecycle/reporting status becomes inconsistent depending on whether the last source video was removed through request actions (`cancelled`) or through library deletion (stays non-cancelled).

Recommendation:
After detaching a source video, if `requestedVideo.videos` becomes empty, set the request status to `cancelled`, clear fields that depend on active workflow (for example `adminAcceptedAt`), and append a consistent history event for auditability. Reuse the same terminal-state behavior already implemented in source-removal flow to keep state transitions deterministic.

**PR comment (line 130):**  
When we detach a deleted library video from edit requests, we only mutate the source array and save. If this was the last source slot, the request can remain in an active status with no resolvable media, which creates inconsistent lifecycle behavior versus the `remove_source_video` path (that now cancels). Can we align this path to transition to `cancelled` (and record history) when no source videos remain?

**Re-verification (task complete):** ✅ Fixed — `detachVideoFromUserEditRequests` cancels with history when `remaining === 0` (lines 123-136). Wired from `EditRequestController.deleteUploadedVideo` and `VideoController.delete` (lazy import). `cancelled` added to constants, types, status resolution, and `ACTION_GUARD.reject`. `applyRemoveSourceVideo` cancels on last source removal; history type switches to `CANCELLED` when removal cancels (lines 1437-1442).
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Deleting a library video leaves orphaned active edit requests | HIGH | ✅ Fixed | `src/services/edit-request.service.ts` | 123-136 |

**Positive notes:** Clean layering (controller → service → repository + utils). `detachSourceVideoReference` keeps library-delete detach separate from athlete `remove_source_video` workflow. Joi list filters pick up `cancelled` via `EDIT_REQUEST_STATUS_VALUES`.

**Merge readiness:** No open backend Critical/High blockers.
