# Backend PR Review — skillshow (`SKSH-271`)

**Repo:** skillshow  
**Branch:** `SKSH-271`  
**Base:** `main...HEAD` (35 files, ~2471 lines)  
**Re-verified:** 2026-06-05 — @ `origin/SKSH-271` → `69ea288`  
**Scope:** DRY/KISS, global consistency, layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical & High only)  
**Findings:** 3 (0 Critical, 3 High) — **0 Open**, **1 Fixed**, **2 Accepted**

---

---
Platform event patch returns 403 as HTTP 500 (`PATCH /v1/videos/:id`)

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/controllers/video.controller.ts  
Lines: 330-333

**Description (original):**  
`applyPlatformEventFromVideoPatch` can throw operational **403**. The `catch` block only mapped **400** and **404**; **403** fell through to `internalError`.

**Re-verification:**  
`respondIfAppError(res, err)` maps **403** (commit `0dcb7bf`). Test: `returns 403 when platform event assignment is forbidden`.

**PR comment:**  
Resolved — `respondIfAppError` returns 403 for share-forbidden platform event assignment.

---

---
Linked-athlete video PATCH maps platform-event errors to HTTP 500

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/controllers/athlete.controller.ts  
Lines: 247-249

**Also:** `src/services/athlete.service.ts` 764-769

**Description:**  
`patchLinkedVideo` calls `applyPlatformEventFromVideoPatch`, but `updateLinkedVideo` uses `handleError()` → platform-event **403/400** may surface as **500**.

**Accepted reason:** Deferred per review decision — not blocking SKSH-271 merge; track as follow-up.

---

---
New event-video queries lack index on `libraryEventSlug`

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/models/video.model.ts  
Lines: ~151

**Consumers:** `src/repositories/video-library.repository.ts` 175, 224, 251, 276

**Description:**  
New aggregates/mutations `$match` on `libraryEventSlug` without a DB index.

**Accepted reason:** Deferred per review decision — index can ship in a follow-up before scale.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Platform event patch 403 surfaced as 500 (`/videos/:id`) | HIGH | ✅ Fixed | `src/controllers/video.controller.ts` | 330-333 |
| 2 | Linked-athlete video PATCH maps platform-event errors to 500 | HIGH | Accepted | `src/controllers/athlete.controller.ts` | 247-249 |
| 3 | New event-video queries lack `libraryEventSlug` index | HIGH | Accepted | `src/models/video.model.ts` | ~151 |

**Positive notes:** Clean controller → service → repo layout for event-athlete/video. RBAC + Joi on admin event routes. `video-user-scope.utils.ts` centralizes My Videos scope. Event video search uses `containsInsensitive`.

**Skipped (per request):** Pagination-related issues; full-review findings #2–3 accepted for this ticket.

**Not reported (pre-existing):** Raw `$regex` on `name` / `q` in `src/utils/video.utils.ts`.

**Merge readiness:** ✅ Ready — no open findings (accepted items deferred).
