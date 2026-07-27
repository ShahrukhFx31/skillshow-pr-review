# PR review — skillshow #249 (SKSH-398)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-398 → main  
**Head:** `947a3e68c1a37019ebc0968704a56cec1f841a18`  
**Scope:** Admin/super_admin fallback in `findOwnedVideoKey` for shared Video Library playback  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Updated:** 2026-07-27 — re-verify; finding #1 fixed (`...VIDEO_LIBRARY_LIST_MATCH`)

## GitHub comments

_(none — no Open Critical/High)_

## Findings

---
Admin library playback filter should reuse VIDEO_LIBRARY_LIST_MATCH

Risk Level: HIGH
File Path: src/services/video.service.ts
Lines: 536-545

Description:
**DRY / Global consistency / Contract.** The admin fallback previously hand-built `{ inVideoLibrary, libraryStatus, uploadStatus }` while the established library match is `VIDEO_LIBRARY_LIST_MATCH` (includes `librarySeq: { $exists: true }`).

**Re-verify:** Head now spreads `...VIDEO_LIBRARY_LIST_MATCH` and imports from `video-library.constants.ts`.

Impact:
- (Resolved) Admins no longer resolve play keys for non-sequenced library-flagged rows outside the shared match.

Recommendation:
_(done)_
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Admin library playback filter should reuse VIDEO_LIBRARY_LIST_MATCH | HIGH | ✅ Fixed | src/services/video.service.ts | 536-545 |

**Merge readiness:** No open Critical/High blockers.
