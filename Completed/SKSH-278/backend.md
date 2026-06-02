# SKSH-278 — Backend review (`skillshow`)

**Branch:** `sksh-278`  
**Base:** `main`  
**Re-review:** `71ca555` (HEAD; no changes since initial feature commit)  
**Scope:** `DELETE /api/v1/upload/profile-image`; team `logoKey` clear on coach PATCH.

## Overview

Adds profile image delete endpoint with Joi validation, `ProfileService.deleteProfileImage`, `UserRepository.clearProfileImageKey`, and coach team update support for empty `logoKey`. Controller tests cover auth, full delete, and orphan-key delete.

**Final re-review:** No Critical or High findings. Layering, validation, prefix checks, and tests align with skillshow conventions. Frontend PATCH-first logo removal uses existing APIs correctly.

---

## Positive notes

- **Layer split:** Route → `validate(profileImageDeleteBodySchema)` → controller → service → repository/S3.
- **Security:** Orphan `{ key }` requires `profile-images/{userId}/` prefix; route under authenticated `/api/v1/*`.
- **Team logo clear:** `logoKey !== undefined` (including `""`) in `coach-link.service` / controller.
- **Tests:** `upload.profile-image.test.ts` for `deleteProfileImage`.
- **Audit:** Full profile delete records `recordAccountGeneralLastUpdatedBy` after clearing `profileImageKey`.

---

## Findings

No **Critical** or **High** findings.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

*No Critical/High findings.*

**Merge readiness:** No open Critical/High blockers.
