# Backend PR Review — skillshow (`SKSH-445`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/256  
**Branch:** `SKSH-445` → main  
**Head:** `c27576a925384d1215064cad2ae05bbd857a9748`  
**Scope:** SkillShow-sourced video visibility-only PATCH enforcement, per-athlete Video Library visibility, thumbnail/metadata route guards, AppError handling  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** `pr-review/Completed/SKSH-445/frontend.md` (admin-ui #367)  
**Updated:** 2026-07-30 — re-review on latest head (expanded library visibility; no new findings)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:**
- Centralized `video-skillshow-edit.utils.ts` with `isSkillshowSourcedVideo` + `assertSkillshowAthleteEditAllowed`.
- **DRY:** `SKILLSHOW_VIDEO_ATHLETE_PATCH_ALLOWED_KEYS` constant; `CLEARABLE`-style guards on metadata mutations.
- Per-athlete Video Library visibility via `resolveViewerVideoIsPublic` / `setLibraryAthleteAssignmentIsPublic`; controller passes `viewerUsername` into patch + response mapping.
- Guards applied at controller (raw body), service entry points (patch, thumbnail, platform metadata), and `respondIfAppError` so 403 surfaces correctly.
- Solid unit/controller test coverage for allow/reject paths and library visibility.
- Frontend #367 skips metadata PATCH in `DistributeModal` for SkillShow-sourced videos — integration aligned.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** Approve for merge — no open Critical/High/Medium findings.
