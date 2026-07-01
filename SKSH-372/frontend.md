# PR review (SKSH-372) — skillshow-admin-ui

PR: `https://github.com/SkillshowFx/skillshow-admin-ui/pull/323`  
Base: `main`  
Head: `SKSH-372` @ `fe5abe1`

Prompt: `pr-review/prompts/frontend-system-prompt.md`

**Aligned with:** [backend.md](./backend.md)

## GitHub comments

### `src/pages/dashboard/athlete/index.tsx`
- **MEDIUM** — Inline `isEditUploaded` filter duplicates shared util (line 90)

## Findings

---
Inline `isEditUploaded` filter duplicates shared util

Risk Level: MEDIUM  
File Path: src/pages/dashboard/athlete/index.tsx  
Lines: 89-91

Description:
**DRY.** The athlete dashboard recent-videos query adds `.filter((v) => v.isEditUploaded !== true)` inline. The same exclusion already lives in `filterVideosByUploadSource` (`src/pages/videos/utils/video-list.utils.ts`, line 13), which documents client-side fallback when list APIs omit upload-source filtering.

Impact:
- If the `isEditUploaded` rule changes, this dashboard path can drift from My Videos tab filtering.
- Minor maintainability debt in a cross-stack ticket that is otherwise aligning counts with backend scope.

Recommendation:
Extract a small shared helper in `video-list.utils.ts`, e.g. `excludeEditUploadedVideos(items: BackendVideo[])`, and use it here (and optionally inside `filterVideosByUploadSource`). The dashboard should not reimplement the flag check inline.

---

PR comment (inline, 2–4 sentences):
This inline `isEditUploaded` check duplicates the exclusion already in `filterVideosByUploadSource` (`video-list.utils.ts`). Extract a shared `excludeEditUploadedVideos` helper and use it here so the dashboard stays aligned if the rule changes. Backend PR #224 already scopes `GET /v1/videos` — this client filter is fine as defense-in-depth once centralized.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Inline `isEditUploaded` filter duplicates shared util | MEDIUM | Open | `src/pages/dashboard/athlete/index.tsx` | 89-91 |

**Merge readiness:** No open Critical/High blockers. One optional MEDIUM DRY cleanup.
