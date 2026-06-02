# Backend PR Review — skillshow (`SKSH-273`)

**Repo:** skillshow  
**Branch:** `SKSH-273`  
**Base:** `main...HEAD`  
**Scope:** Layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical and High only)  
**Findings:** 1 (0 Critical, 1 High) — **re-verified: ✅ Fixed**

---

---
Public share treats `isPublic` as “not false” while upload UI treats only `true` as public

Risk Level: HIGH
File Path: src/utils/video-share.utils.ts
Lines: 5-7

Description:
`isVideoPubliclyShareable` returned `true` when `isPublic` was `undefined` (`isPublic !== false`), while the upload UI only treated `isPublic === true` as public.

Impact:
- Athletes could believe a video was private while share URLs still worked
- Inconsistent API vs UI contract

Recommendation:
Require explicit `isPublic === true`; default new videos to private; update tests.

**PR comment (line 6):**  
**High:** `isVideoPubliclyShareable` treats missing/`undefined` `isPublic` as public while the new upload UI only enables sharing when `isPublic === true`. Please align backend share eligibility with explicit `true`.

**Re-verification (commit `64842b2`):** ✅ Fixed — `isVideoPubliclyShareable` now returns `isPublic === true`; `VIDEO_PUBLIC_ONLY_QUERY` is `{ isPublic: true }`; model default `isPublic: false`; tests updated.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Public share treats `isPublic` as “not false” vs UI `=== true` | HIGH | ✅ Fixed | `src/utils/video-share.utils.ts` | 5-7 |

**Positive notes:** Clean layering (routes → controller → `VideoPublicService` → repository). Params validated with `videoIdParamSchema`. Public route outside `/v1` auth. Role-scoped profile via `toPublicVideoShareProfile`. Good unit coverage.

**Merge readiness:** No open backend Critical/High blockers.
