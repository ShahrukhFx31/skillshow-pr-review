# Frontend PR Review — skillshow-admin-ui (`SKSH-446`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/369  
**Branch:** `SKSH-446` → main  
**Head:** `831fdf3a3409de6a15decbc2c368a67d501b060a`  
**Scope:** Edit-request upload — truncate filename-derived video title to backend max (100) and avoid duplicate error toasts  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

### `src/pages/editRequest/components/UploadNewVideoModal.tsx` — line 203

**MEDIUM** — Upload error toast guard uses `err.message` as “already notified” proxy

The catch block skips the generic toast whenever `err instanceof Error` has a non-empty `message`, but that is not equivalent to apiClient having shown a user-facing toast. Axios rejections always carry a generic message (e.g. `Request failed with status code 400`) even when `isDisplayMessage` is false and no toast was shown. `putToPresignedUrl` also rejects with `Error("Upload failed: …")` / `Error("Network error")` without going through apiClient, so the generic toast is suppressed for S3 upload failures too. Inline `uploadError` still renders, but users lose the supplementary toast on those paths.

## Findings

---
Upload error toast guard uses `err.message` as “already notified” proxy

Risk Level: MEDIUM
File Path: src/pages/editRequest/components/UploadNewVideoModal.tsx
Lines: 202-205

Description:
The new catch logic assumes a non-empty `err.message` means apiClient already surfaced the failure. Axios errors always include a message; apiClient only toasts HTTP error bodies when `isDisplayMessage === true` (see `apiClient.ts` interceptor). Presigned S3 upload failures from `putToPresignedUrl` throw plain `Error` objects with messages but never pass through apiClient.

Impact:
- Duplicate toasts are correctly avoided when apiClient shows an `isDisplayMessage` error, but other failures (presigned upload, API errors without `isDisplayMessage`) no longer get the generic toast even though apiClient did not notify the user.
- Inline `uploadError` still shows in the modal row, so the regression is reduced toast visibility rather than a fully silent failure.

Recommendation:
Gate the generic toast on whether apiClient actually displayed the error, not on `err.message` presence. For example, use `axios.isAxiosError(err)` and `getServerResponseMeta(err.response?.data)` — skip the generic toast only when `isDisplayMessage && message`. For non-Axios errors (presigned upload), keep `toast.error(\`Failed to upload ${file.name}.\`)`.
---

**Positive notes:** `EDIT_REQUEST_VIDEO_TITLE_MAX_LENGTH = 100` aligns with backend Joi (`edit-request.validation.ts`, `video.validation.ts`). Truncating in `stripFileTitle` prevents validation failures on long filenames. Protected modules untouched.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Upload error toast guard uses `err.message` as “already notified” proxy | Medium | Open | src/pages/editRequest/components/UploadNewVideoModal.tsx | 202-205 |

**Merge readiness:** One open Medium finding — no Critical/High blockers; safe to merge if toast heuristic is accepted for this release.
