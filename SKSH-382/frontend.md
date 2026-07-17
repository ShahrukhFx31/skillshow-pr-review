# PR review (SKSH-382) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#346](https://github.com/SkillshowFx/skillshow-admin-ui/pull/346) |
| Branch | `SKSH-382` → `main` |
| Head | `e53fb9fb24af5c48d1eaa1d9f13d14e42ac4f98c` |
| Scope | Video Library bulk-upload wizard; concurrent upload pool; dropzone extensions |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

### Protected modules

| Module | Status |
|--------|--------|
| Pagination / audit / destructive-confirm protected modules | **Not modified** ✅ |

### Positive notes

- Register mapping correctly uses `parsed.eventSlug` and `overwriteVideoId` for confirmed duplicates.
- Wizard flow (select → review → upload) and summary cards are clear.
- `UploadDropzone` extensions remain backward-compatible.

## GitHub comments

Posted: REQUEST_CHANGES on `e53fb9fb` (retry after prior 422 — `retryUpload` not in diff).

### `src/pages/videoLibrary/dashboard/components/video-library-upload-modal.tsx`
- **L120** — Validation failure desyncs `selectedFiles` from `validation` (HIGH)

### `src/pages/videos/hooks/useVideoUpload.ts`
- **L171** (`enqueueUpload`) — `retryUpload` bypasses concurrency pool (HIGH); `retryUpload` L677–693 not in diff

### `src/pages/videoLibrary/dashboard/components/video-library-bulk-upload-review.tsx`
- **L16** — Duplicated `SummaryCard` markup (MEDIUM)

## Findings

---
Validation failure desyncs file selection state

Risk Level: HIGH
File Path: src/pages/videoLibrary/dashboard/components/video-library-upload-modal.tsx
Lines: 106-121

Description:
**Contract** — `handleFilesSelected` commits `selectedFiles` before `runValidation` finishes. On catch, only a toast runs; stale `validation` / accumulated files remain.

Impact:
- Review/upload can disagree with the selected File list by index
- Re-selection after error duplicates files in the batch

Recommendation:
On catch, call `resetWizard()` (or clear `selectedFiles` + `validation` and stay on `select`). Prefer committing `selectedFiles` only after a successful validate.
---

---
retryUpload bypasses concurrency pool

Risk Level: HIGH
File Path: src/pages/videos/hooks/useVideoUpload.ts
Lines: 677-693

Description:
**Contract** — Bulk upload sets `maxConcurrentUploads: 15` and uses `enqueueUpload`, but `retryUpload` awaits `uploadSingle` / `uploadMultipart` directly.

Impact:
- Retries during a bulk batch can exceed the concurrency cap
- Browser / S3 throttling risk on large batches

Recommendation:
Route retries through `enqueueUpload` (same as `addFilesWithMappings`).
---

---
SummaryCard duplicated across review and summary

Risk Level: MEDIUM
File Path: src/pages/videoLibrary/dashboard/components/video-library-bulk-upload-review.tsx
Lines: 16-54

Description:
**DRY** — Identical `SummaryCard` helper is copy-pasted in `video-library-bulk-upload-summary.tsx`.

Impact:
- Styling/label drift between review and post-upload summary

Recommendation:
Extract one shared summary-card component and reuse.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Validation failure desyncs file selection state | HIGH | Open | `src/pages/videoLibrary/dashboard/components/video-library-upload-modal.tsx` | 106-121 |
| 2 | `retryUpload` bypasses concurrency pool | HIGH | Open | `src/pages/videos/hooks/useVideoUpload.ts` | 677-693 |
| 3 | SummaryCard duplicated across review and summary | MEDIUM | Open | `src/pages/videoLibrary/dashboard/components/video-library-bulk-upload-review.tsx` | 16-54 |

**Merge readiness:** Not ready — 2 open High findings (validate state + retry concurrency).
