# Backend PR Review — skillshow (`SKSH-273`)

**Repo:** skillshow  
**Branch:** `SKSH-273`  
**Base:** `main...HEAD`  
**Scope:** Layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical and High only)  
**Findings:** 1 (0 Critical, 1 High) — **all resolved**

---

---
Public share treats `isPublic` as “not false” while upload UI treats only `true` as public

Risk Level: HIGH
File Path: src/utils/video-share.utils.ts
Lines: 5-7

Description:
`isVideoPubliclyShareable` previously returned `true` when `isPublic` was `undefined` (`isPublic !== false`), while the upload UI only treated `isPublic === true` as public.

Impact:
- Athletes could believe a video was private while share URLs still worked
- Inconsistent API vs UI contract

Recommendation:
Require explicit `isPublic === true`; default new videos to private; update tests.

**Re-verification:** ✅ Fixed — `return isPublic === true`; `VIDEO_PUBLIC_ONLY_QUERY` is `{ isPublic: true }`; model default `isPublic: false`; create/upload paths set `isPublic: false`; tests expect only `true` as shareable.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Public share treats `isPublic` as “not false” vs UI `=== true` | HIGH | ✅ Fixed | `src/utils/video-share.utils.ts` | 5-7 |

**Positive notes:** Clean layering (routes → controller → `VideoPublicService` → repository). Params validated with `videoIdParamSchema`. Public route outside `/v1` auth. Role-scoped profile via `toPublicVideoShareProfile`. Good unit coverage.

**Merge readiness:** No open backend Critical/High blockers.
