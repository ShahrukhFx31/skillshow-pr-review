# PR review (SKSH-382) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#346](https://github.com/SkillshowFx/skillshow-admin-ui/pull/346) |
| Branch | `SKSH-382` → `main` |
| Head | `be2e8779e808b3d92178b8d4f86143a72b893312` |
| Scope | Video Library bulk-upload wizard; concurrent upload pool; dropzone extensions |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Verified | 2026-07-17 vs prior head `e53fb9fb` |

### Protected modules

| Module | Status |
|--------|--------|
| Pagination / audit / destructive-confirm protected modules | **Not modified** ✅ |

### Positive notes

- Register mapping uses `result.eventSlug ?? result.parsed?.eventSlug` and `overwriteVideoId` for confirmed duplicates.
- Wizard commits `selectedFiles` only after validate succeeds.
- Shared `VideoLibraryBulkUploadSummaryCard`; `retryUpload` goes through `enqueueUpload`.

## GitHub comments

No open findings to post (prior REQUEST_CHANGES resolved on head).

## Findings

---
Validation failure desyncs file selection state

Risk Level: HIGH
File Path: src/pages/videoLibrary/dashboard/components/video-library-upload-modal.tsx
Lines: 106-121

Description:
**Contract** — `handleFilesSelected` committed `selectedFiles` before `runValidation` finished.

Impact:
- Review/upload could disagree with the selected File list by index

Recommendation:
Commit `selectedFiles` only after a successful validate.

Status: ✅ Fixed — `setSelectedFiles(files)` only inside validate try after success; catch leaves prior selection unchanged.
---

---
retryUpload bypasses concurrency pool

Risk Level: HIGH
File Path: src/pages/videos/hooks/useVideoUpload.ts
Lines: 677-693

Description:
**Contract** — Bulk upload used `enqueueUpload`, but `retryUpload` awaited uploads directly.

Impact:
- Retries could exceed the concurrency cap

Recommendation:
Route retries through `enqueueUpload`.

Status: ✅ Fixed — `retryUpload` uses `enqueueUpload`; clears `failed` before enqueue so the pool does not skip.
---

---
SummaryCard duplicated across review and summary

Risk Level: MEDIUM
File Path: src/pages/videoLibrary/dashboard/components/video-library-bulk-upload-review.tsx
Lines: 16-54

Description:
**DRY** — Identical `SummaryCard` helper was copy-pasted in summary.

Impact:
- Styling/label drift

Recommendation:
Extract one shared summary-card component.

Status: ✅ Fixed — `video-library-bulk-upload-summary-card.tsx` shared by review + summary.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Validation failure desyncs file selection state | HIGH | ✅ Fixed | `video-library-upload-modal.tsx` | 106-121 |
| 2 | `retryUpload` bypasses concurrency pool | HIGH | ✅ Fixed | `useVideoUpload.ts` | 677-693 |
| 3 | SummaryCard duplicated across review and summary | MEDIUM | ✅ Fixed | `video-library-bulk-upload-review.tsx` | 16-54 |

**Merge readiness:** Ready — prior High/Medium findings fixed on `be2e8779`; no new Critical/High/Medium open.
