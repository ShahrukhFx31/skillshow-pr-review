# Frontend PR Review — skillshow-admin-ui (`SKSH-327`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-327`  
**Base:** `main...HEAD` @ `ee89e57c`  
**Initial review:** 2026-06-09  
**Re-review:** 2026-06-09 (`ee89e57c` — `fix: feedbacks`)  
**Scope:** Linked-athlete `VideoPlayer` play-url context, upload thumbnail DRY refactor, Skill Show tab delete guard, library-upload thumbnail generation (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 2 (0 Critical, 0 High, 2 Medium) — **0 Open**, **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, `AuditLogTable.tsx` | **Not modified** |

`DestructiveActionConfirmModal` remains correctly composed via `use-my-videos-actions.tsx` (unchanged).

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/dashboard/components/VideoCard.tsx` | Pass `relationId`, `variant="fill"` to `VideoPlayer` |
| `src/pages/videos/details/components/EditVideoModal.tsx` | Pass `relationId` for linked edit preview |
| `src/pages/videos/details/components/VideoInfoCard.tsx` | Accept/pass `relationId` |
| `src/pages/videos/details/components/VideoPlayer.tsx` | `getPlayUrlForContext`, `variant` layout |
| `src/pages/videos/details/index.tsx` | Wire `linkedAthleteRelationId` to `VideoInfoCard` |
| `src/pages/videos/details/interfaces/index.ts` | `relationId`, `variant` props |
| `src/pages/videos/hooks/useVideoUpload.ts` | DRY thumbnail helpers; library-upload thumbnails; `resolveLibraryMongoVideoId` |
| `src/pages/videos/my-videos/dashboard/components/my-videos-list-card.tsx` | Hide delete on Skill Show tab |
| `src/pages/videos/my-videos/dashboard/components/my-videos-actions-cell.tsx` | `canDeleteDraft` prop rename |
| `src/pages/videos/my-videos/dashboard/components/my-videos-columns.tsx` | `canDeleteDraft` prop rename |
| `src/pages/videos/my-videos/dashboard/components/my-videos-table.tsx` | `canDeleteDraft` prop rename |
| `src/pages/videos/my-videos/dashboard/components/my-videos-table-responsive.tsx` | `canDeleteDraft` prop rename |
| `src/pages/videos/my-videos/dashboard/hooks/use-my-videos-actions.tsx` | `canDeleteDraft` prop rename |
| `src/pages/videos/my-videos/dashboard/types.ts` | `canDeleteDraft` type rename |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `VideoPlayer` uses `getPlayUrlForContext` (matches `mutations.ts` / backend linked play-url) | ✅ Contract |
| `RecentVideos` spreads `relationId` via `{...video}` into `VideoCard` | ✅ |
| Thumbnail extraction consolidated into `generateAndApplyThumbnails` / `applyGeneratedThumbnails` | ✅ DRY |
| `thumbnailKey` now patched with frames (library + platform) | ✅ Improvement |
| Skill Show tab delete gated with `MY_VIDEOS_SOURCE_TAB.skill` | ✅ Ticket fix |
| Protected table/pagination modules untouched | ✅ |
| Library-upload retry resolves `playbackVideoId` via `resolveLibraryMongoVideoId` | ✅ Fixed — see #1 |
| `canDeleteDraft` prop renamed end-to-end through table/actions chain | ✅ Fixed — see #2 |

### Positive notes

- **Contract:** Linked dashboard/detail playback now hits the linked-athlete play-url API when `relationId` is present — pairs with backend SKSH-327 access fix.
- **DRY:** `useVideoUpload` thumbnail pipeline deduplicated; `applyGeneratedThumbnails` centralizes local state + `patchVideoBackend` for frames/keys/primary thumb.
- **UX:** `variant="fill"` with `object-contain` fixes modal player cropping on parent dashboard cards.
- **Re-review:** Feedback commit addresses both prior Medium findings.

---

## GitHub comments

No open findings — prior comments resolved on branch.

---

## Findings

---
Library-upload retry skips thumbnail generation when `existingBackendId` is set

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/videos/hooks/useVideoUpload.ts  
Lines: 84-96, 255-283, 429-457

Description:
**Contract / edge case.** Library-upload thumbnail generation only ran when `mongoVideoId` was captured during the `!backendId` registration branch. On retry, `existingBackendId` skipped registration and thumbnails were never generated.

**Re-verification (`ee89e57c`):** ✅ **Fixed** — `resolveLibraryMongoVideoId` uses registration `playbackVideoId` when available, otherwise fetches `getVideoLibraryItem(libraryItemId)`. Both `uploadSingle` and `uploadMultipart` library paths call it whenever `backendId` is set.

**PR comment (`useVideoUpload.ts` line 269):** **Resolved** — `resolveLibraryMongoVideoId` correctly resolves `playbackVideoId` on retry via library item lookup.

---

---
`canDeleteDraft` forwarded under misleading `canEditVideo` prop name

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/videos/my-videos/dashboard/components/my-videos-list-card.tsx  
Lines: 61-65, 131

Description:
**KISS / maintainability.** `canDeleteDraft` was passed through the table/actions chain as `canEditVideo`, obscuring delete-only gating on the Skill Show tab.

**Re-verification (`ee89e57c`):** ✅ **Fixed** — prop renamed to `canDeleteDraft` in `types.ts`, `use-my-videos-actions.tsx`, `my-videos-table.tsx`, `my-videos-columns.tsx`, `my-videos-actions-cell.tsx`, `my-videos-table-responsive.tsx`, and `my-videos-list-card.tsx`.

**PR comment (`my-videos-list-card.tsx` line 65):** **Resolved** — `canDeleteDraft` is now consistent end-to-end.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Library-upload retry skips thumbnail generation when `existingBackendId` is set | MEDIUM | ✅ Fixed | src/pages/videos/hooks/useVideoUpload.ts | 84-96, 255-283 |
| 2 | `canDeleteDraft` forwarded under misleading `canEditVideo` prop name | MEDIUM | ✅ Fixed | src/pages/videos/my-videos/dashboard/ (types, hooks, components) | — |

**Merge readiness:** No open Critical/High/Medium blockers. All findings fixed on `ee89e57c`. Ready to merge.
