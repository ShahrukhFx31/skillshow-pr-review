# Backend PR Review — skillshow (`SKSH-267`)

**Base:** `main...HEAD`  
**Branch:** `sksh-267-main`  
**Re-verified:** 2026-05-26  

**Changed files:** `BaseController.ts`, `video.constants.ts`, `video.controller.ts`, `app-error.ts`, `distribution-job.repository.ts`, `vendor-log.repository.ts`, `video-upload.service.ts`, `video.controller.test.ts`

| Category | Fixed | Open |
|----------|-------|------|
| High (original) | 2 | 0 |

**Regression re-check:** **0 new** Critical or High issues in PR diff.

---

## Service bypasses repository layer for distribution checks

**Status:** ✅ **FIXED**

**Risk Level:** HIGH  
**File Path:** `src/services/video-upload.service.ts`  
**Lines:** 94-102

**Verification:**  
`vendorLogRepository.existsCompletedForVideo` → `distributionJobRepository.findAllIdsByVideoId` + `existsCompletedForDistributionJobIds`. No direct model usage in the service.

---

## Magic-string errors for domain failures

**Status:** ✅ **FIXED**

**Risk Level:** HIGH  
**File Path:** `src/controllers/video.controller.ts`  
**Lines:** 429-432

**Verification:**  
`NotFoundError` / `BadRequestError` from service; `respondIfAppError(res, err)` in delete handler. Tests use `NotFoundError` for 404 path.

---

## Summary

| # | Title | Risk | Status |
|---|--------|------|--------|
| 1 | Service bypasses repositories | HIGH | ✅ Fixed |
| 2 | Magic-string error mapping | HIGH | ✅ Fixed |

---

## Regression re-check (PR diff only)

**No new Critical or High issues** in changed files.

**Follow-up (not in this PR’s diff, not counted as open):**

- `edit-request.controller.ts` still uses legacy `error.message === "NOT_FOUND"` while calling `deleteUploadedVideo` — separate PR if that route must return 404/400 via `AppError`.
- No test in this PR for `BadRequestError` / `HAS_PUBLISHED_DISTRIBUTION` on `VideoController.delete` (404 path is covered).

**All tracked backend issues in this PR are resolved.**
