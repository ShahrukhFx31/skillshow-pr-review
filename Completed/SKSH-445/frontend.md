# Frontend PR Review — skillshow-admin-ui (`SKSH-445`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/367  
**Branch:** `SKSH-445` → main  
**Head:** `2e3f70a8bfeb93d1c5cfe393b5dc03319de7f2ac`  
**Scope:** SkillShow-sourced video visibility-only edit UI, action menu labels/tooltips, `isSkillshowUploadedSource` utility, distribute flow guard  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/Completed/SKSH-445/backend.md` (skillshow #256)  
**Updated:** 2026-07-30 — re-review on latest head (#1 still fixed; no new findings)

## GitHub comments

_(No open inline comments — all prior findings resolved.)_

## Findings

---
DistributeModal still PATCHes metadata before distribute — breaks SkillShow-sourced distribution

Risk Level: HIGH
File Path: src/pages/videos/details/components/distribute/DistributeModal.tsx
Lines: 275-303, 355-358

Description:
**Global consistency / Contract.** Earlier head still called `updateVideoMutation` with title/vendorMetadata before distribute. Latest head gates the pre-flight PATCH with `if (!isSkillshowVisibilityOnly)` and disables metadata reset for SkillShow-sourced videos.

Impact:
- Resolved — distribute proceeds without 403 pre-flight PATCH; metadata reset blocked for visibility-only videos.

Recommendation:
N/A — fixed on latest head.
---

**Positive notes:** Shared `isSkillshowUploadedSource` utility reused in list filter, distribute enablement, and action menus; `EditVideoModal` visibility-only path sends `{ isPublic }` only; tooltips/labels centralized in `EDIT_VIDEO_MODAL_COPY` and `VIDEO_LIST_ACTION_LABEL`; `MobileVideoCard` and `MyVideosActionsCell` both pass `editVisibilityOnly`; `TITLE_MAX` replaces local duplicate in edit modal.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | DistributeModal still PATCHes metadata before distribute — breaks SkillShow-sourced distribution | HIGH | ✅ Fixed | src/pages/videos/details/components/distribute/DistributeModal.tsx | 275-303, 355-358 |

**Merge readiness:** No open Critical/High/Medium blockers — approve for merge.
