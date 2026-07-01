# PR review (SKSH-372) — skillshow-admin-ui

PR: `https://github.com/SkillshowFx/skillshow-admin-ui/pull/323`  
Base: `main`  
Head: `SKSH-372` @ `2821c25`  
**Re-review:** 2026-07-01 @ `2821c25` (was `fe5abe1`)

Prompt: `pr-review/prompts/frontend-system-prompt.md`

**Aligned with:** [backend.md](./backend.md)

## GitHub comments

No new inline comments — prior finding resolved in `2821c25`.

## Findings

---
Inline `isEditUploaded` filter duplicates shared util

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/dashboard/athlete/index.tsx  
Lines: 86-89 (was 89-91)

Description:
**DRY.** The athlete dashboard recent-videos query added `.filter((v) => v.isEditUploaded !== true)` inline. The same exclusion already lived in `filterVideosByUploadSource`.

**Re-review evidence:** `2821c25` extracts `excludeEditUploadedVideos` in `video-list.utils.ts`, uses it in the dashboard (`excludeEditUploadedVideos(await listVideos(3))`), and refactors `filterVideosByUploadSource` to call the shared helper.

Impact (was):
- Dashboard path could drift from My Videos tab filtering if the rule changed.

Recommendation (was):
Extract `excludeEditUploadedVideos` — **done.**

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Inline `isEditUploaded` filter duplicates shared util | MEDIUM | ✅ Fixed | `src/pages/videos/utils/video-list.utils.ts` | 8-11 |

**Merge readiness:** No open Critical/High/Medium blockers.
