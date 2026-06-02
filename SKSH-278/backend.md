# SKSH-278 — Backend review (`skillshow`)

**Branch:** `sksh-278`  
**Base:** `main`  
**Re-review:** `849b3da8` (frontend-only; backend unchanged)  
**Scope:** Profile image delete API; team `logoKey` clear on update.

## Overview

Adds `DELETE /api/v1/upload/profile-image` with Joi validation, `ProfileService.deleteProfileImage`, and `UserRepository.clearProfileImageKey`. Team logo removal is supported via empty `logoKey` on coach team PATCH.

**Re-review:** No backend changes since initial review. Frontend fixes use existing endpoints correctly.

---

## Positive notes

- **Layer split:** Route → validate → controller → service → repository/S3.
- **Security:** Orphan `{ key }` deletes require `profile-images/{userId}/` prefix; authenticated `/api/v1/*`.
- **Team logo clear:** Empty `logoKey` supported on PATCH.
- **Tests:** Controller tests for delete profile image (401, full delete, orphan key).

---

## Findings

No **Critical** or **High** findings. No new issues on re-review.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

*No Critical/High findings.*
