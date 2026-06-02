# Backend PR Review — skillshow (`SKSH-271`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-271`  
**Base:** `main...HEAD`  
**Re-reviewed:** 2026-06-01 (after developer fixes)  
**Scope:** Layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical & High only)  
**Findings:** 1 (0 Critical, 1 High) — **0 Open**, **1 Fixed**

---

---
Platform event patch returns 403 as HTTP 500

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/controllers/video.controller.ts  
Lines: 330-333

**Description (original):**  
`PATCH /v1/videos/:id` calls `applyPlatformEventFromVideoPatch`, which can throw operational **403**. The `catch` block only mapped **400** and **404**; **403** fell through to `internalError`.

**Re-verification:**  
Commit `0dcb7bf` replaces the manual status branching with `respondIfAppError(res, err)`, which maps **403** via `BaseController.respondIfAppError` (both `AppError` and legacy `throwError` shapes). Test added: `returns 403 when platform event assignment is forbidden` in `tests/controllers/video.controller.test.ts`.

**PR comment (if still open on old diff):**  
Resolved — `respondIfAppError` now returns 403 for share-forbidden platform event assignment.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Platform event patch 403 surfaced as 500 | HIGH | ✅ Fixed | `src/controllers/video.controller.ts` | 330-333 |

**Positive notes:** Event-athlete/video routes use Joi `validate()` + RBAC `authorize`. Event controllers use `next(err)`. Event video search uses `containsInsensitive`. `GET /v1/events/me` returns all active events for the upload picker (documented in swagger). `countAthletesByEvents` batches registrations and video-library usernames.

**New issues on re-review:** None (Critical/High).

**Skipped (per request):** Pagination-related issues (`listForEvent` in-memory slice, `listAvailableForEvent` user search before slice).

**Not reported (pre-existing):** Raw `$regex` on `name` / `q` in `src/utils/video.utils.ts` — unchanged in this PR.
