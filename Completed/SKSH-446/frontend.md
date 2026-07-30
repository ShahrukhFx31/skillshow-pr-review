# Frontend PR Review — skillshow-admin-ui (`SKSH-446`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/369  
**Branch:** `SKSH-446` → main  
**Head:** `78f92c020f4d4d743ad52c25e8955f2f77209a92`  
**Scope:** Edit-request upload — truncate filename-derived video title to backend max (100) and avoid duplicate error toasts  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** _(none on branch `SKSH-446`)_  
**Updated:** 2026-07-30 — re-review on latest head (#1 fixed)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

---
Upload error toast guard uses `err.message` as “already notified” proxy

Risk Level: MEDIUM
File Path: src/pages/editRequest/components/UploadNewVideoModal.tsx
Lines: 204-214

Description:
The catch block previously skipped the generic toast whenever `err.message` was non-empty, which did not match apiClient toast behavior.

Impact:
- Resolved — latest head gates on `isAxiosError` + `response.data.isDisplayMessage === true` with a non-empty message; generic toast still shows for presigned/S3 and Axios errors without `isDisplayMessage`.

Recommendation:
N/A — fixed on latest head.
---

**Positive notes:** `EDIT_REQUEST_VIDEO_TITLE_MAX_LENGTH = 100` aligns with backend Joi; filename truncation in `stripFileTitle` prevents validation failures; toast logic now matches apiClient contract. Protected modules untouched.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Upload error toast guard uses `err.message` as “already notified” proxy | Medium | ✅ Fixed | src/pages/editRequest/components/UploadNewVideoModal.tsx | 204-214 |

**Merge readiness:** No open Critical/High/Medium blockers — approve for merge.
