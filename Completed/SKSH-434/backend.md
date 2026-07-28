# Backend PR Review — skillshow (`SKSH-434`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/252  
**Branch:** `SKSH-434`  
**Base:** `main`  
**Head:** `23e603072beddaa1dfbdc3332505cf582e9993d7`  
**Scope:** Public share eligibility for SkillShow / Video Library uploads  
**Prompts:** `pr-review/prompts/backend-system-prompt.md`  
**Updated:** 2026-07-28 — re-verify @ `23e6030` (`refactor: enhance public video share logic and user profile resolution`)

**Findings:** 2 (0 Critical, 2 High) — **0 Open**, **2 ✅ Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| list-query / audit-log / change-stream stack | **Not modified** ✅ |

---

## GitHub comments

_(none — no Open findings)_

---

## Findings

---
Public share query drops Video Library visibility guards

Risk Level: HIGH  
**Status:** ✅ Fixed @ `23e6030`  
File Path: src/constants/video.constants.ts  
Lines: 40-48

Evidence:
`VIDEO_PUBLIC_SHARE_QUERY` is now `{ isPublic: true, $or: [{ inVideoLibrary: { $ne: true } }, VIDEO_SKILL_SHOW_TAGGED_MATCH] }` (excludes soft-deleted / failed library rows). Profile filter uses `$and` with `buildMyVideosListScopeQuery`. Repository tests cover `findPublicShareLean` / `incrementShareViewCount`.
---

---
Library public share resolves admin uploader profile, not tagged athlete

Risk Level: HIGH  
**Status:** ✅ Fixed @ `23e6030`  
File Path: src/services/video-public.service.ts  
Lines: 25-75

Evidence:
`resolvePublicShareProfileUserId` uses `libraryTaggedAthleteUsernames` + `findLeanActiveUsersByUsernames`, falls back to `row.user` only when untagged. Lean select includes `inVideoLibrary` / `libraryAthleteUsernames`. Service test asserts athlete profile, not admin uploader.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Public share query drops Video Library visibility guards | HIGH | ✅ Fixed | src/constants/video.constants.ts | 40-48 |
| 2 | Library public share resolves admin uploader profile, not tagged athlete | HIGH | ✅ Fixed | src/services/video-public.service.ts | 25-75 |

**Merge readiness:** Approve for merge — no open Critical/High findings.
