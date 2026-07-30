# Frontend PR Review — skillshow-admin-ui (`SKSH-435`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/364  
**Branch:** `SKSH-435` → main  
**Head:** `a61217c560ddf25be3f5f5db9b94dde2959e236e`  
**Scope:** Allow complimentary SkillShow delivery + additional athlete videos in edit-request create UX (lock free-edit, pricing, selection)  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-435/backend.md` (skillshow #253)  
**Updated:** 2026-07-30 — re-review on latest head (#1 fixed)

## GitHub comments

_(No open inline comments — prior finding resolved.)_

## Findings

---
Duplicated free-edit lock merge logic across three handlers

Risk Level: MEDIUM  
File Path: src/pages/editRequest/index.tsx  
Lines: 247, 321, 330

Description:
**DRY.** Earlier head duplicated lock/merge rules in `handleStartWithVideos`, `UploadVideosScreen.onSelectVideos`, and `EnhanceVideoScreen.onAddVideos`. Latest head extracts `mergeEditRequestVideoSelection`, `editRequestVideoLockId`, and `isEditRequestFreeEditVideo` in `edit-request-video-selection.utils.ts`; modal imports the same helpers.

Impact:
- Resolved — selection invariants centralized; handlers and modal share one util.

Recommendation:
N/A — fixed on latest head.
---

**Positive notes:** Pricing uses `paidVideoCount` / `isFullyComplimentary`; complimentary row remove blocked; selection mode stays `multiple`; backend payment contract now aligned.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Duplicated free-edit lock merge logic across three handlers | MEDIUM | ✅ Fixed | src/pages/editRequest/index.tsx | 247, 321, 330 |

**Merge readiness:** No open Critical/High/Medium blockers.
