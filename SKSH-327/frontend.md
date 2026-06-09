# Frontend PR Review — skillshow-admin-ui (`SKSH-327`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-327`  
**Base:** `main...HEAD` @ `25fd0736`  
**Initial review:** 2026-06-09  
**Scope:** Linked-athlete `VideoPlayer` play-url context, upload thumbnail DRY refactor, Skill Show tab delete guard, library-upload thumbnail generation (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 2 (0 Critical, 0 High, 2 Medium) — **2 Open**

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
| `src/pages/videos/hooks/useVideoUpload.ts` | DRY thumbnail helpers; library-upload thumbnails |
| `src/pages/videos/my-videos/dashboard/components/my-videos-list-card.tsx` | Hide delete on Skill Show tab |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `VideoPlayer` uses `getPlayUrlForContext` (matches `mutations.ts` / backend linked play-url) | ✅ Contract |
| `RecentVideos` spreads `relationId` via `{...video}` into `VideoCard` | ✅ |
| Thumbnail extraction consolidated into `generateAndApplyThumbnails` / `applyGeneratedThumbnails` | ✅ DRY |
| `thumbnailKey` now patched with frames (library + platform) | ✅ Improvement |
| Skill Show tab delete gated with `MY_VIDEOS_SOURCE_TAB.skill` | ✅ Ticket fix |
| Protected table/pagination modules untouched | ✅ |
| Library-upload retry thumbnail gap when `existingBackendId` set | ⚠️ See #1 |
| `canDeleteDraft` forwarded as `canEditVideo` through table chain | ⚠️ See #2 |

### Positive notes

- **Contract:** Linked dashboard/detail playback now hits the linked-athlete play-url API when `relationId` is present — pairs with backend SKSH-327 access fix.
- **DRY:** `useVideoUpload` thumbnail pipeline deduplicated; `applyGeneratedThumbnails` centralizes local state + `patchVideoBackend` for frames/keys/primary thumb.
- **UX:** `variant="fill"` with `object-contain` fixes modal player cropping on parent dashboard cards.

---

## GitHub comments (Open findings)

| # | File (inline comment anchor) | PR comment line |
|---|------------------------------|-----------------|
| 1 | `src/pages/videos/hooks/useVideoUpload.ts` | 260 |
| 2 | `src/pages/videos/my-videos/dashboard/components/my-videos-list-card.tsx` | 65 |

---

## Findings

---
Library-upload retry skips thumbnail generation when `existingBackendId` is set

Risk Level: MEDIUM
File Path: src/pages/videos/hooks/useVideoUpload.ts
Lines: 237-262, 408-431

Description:
**Contract / edge case.** Library-upload thumbnail generation only runs when `mongoVideoId` is captured during the `!backendId` registration branch. On `retryUpload`, `uploadSingle` / `uploadMultipart` pass `v.backendVideoId` as `existingBackendId`, so registration is skipped and `mongoVideoId` stays `undefined` — `generateAndApplyThumbnails` is never called. Platform-upload retry has the same pattern but already used the video document id as `backendId`; library items use a separate library id vs `playbackVideoId`.

Impact:
- Retry after a successful S3 upload but failed/skipped thumbnail step leaves library rows without generated frames/`thumbnailKey`.
- User must fully remove and re-upload (losing `backendVideoId`) to trigger registration + thumbnails again.

Recommendation:
Resolve `playbackVideoId` when `isLibraryUpload && backendId && !mongoVideoId` — e.g. store `playbackVideoId` on `UploadVideo` at registration time, or fetch `getVideoLibraryItem(backendId)` before thumbnail generation:

```typescript
let mongoVideoId: string | undefined;
if (isLibraryUpload && backendId && !mongoVideoId) {
  const item = await getVideoLibraryItem(backendId);
  mongoVideoId = item.playbackVideoId ?? undefined;
}
if (mongoVideoId) {
  await generateAndApplyThumbnails(id, mongoVideoId, URL.createObjectURL(file), { revokeUrl: true });
}
```

**PR comment (`useVideoUpload.ts` line 260):** **Medium:** Library-upload retry passes `existingBackendId`, so `mongoVideoId` is never set and thumbnails are skipped after a failed thumbnail step. Resolve `playbackVideoId` from the library item (or persist it on upload state) before calling `generateAndApplyThumbnails`.

---

---
`canDeleteDraft` forwarded under misleading `canEditVideo` prop name

Risk Level: MEDIUM
File Path: src/pages/videos/my-videos/dashboard/components/my-videos-list-card.tsx
Lines: 61-65, 131

Description:
**KISS / maintainability.** `my-videos-list-card.tsx` correctly computes `canDeleteDraft`, but passes it to `useMyVideosActions` and `MyVideosTable` as `canEditVideo`. Downstream (`my-videos-actions-cell.tsx`, `my-videos-columns.tsx`) the prop only gates draft **delete**, not edit/view/retry. Future readers may assume broader edit permissions are disabled on the Skill Show tab.

Impact:
- Risk of regressions if a contributor wires new edit actions to `canEditVideo` expecting full edit capability on the athlete tab.
- Naming drift between `canDeleteDraft` (local) and `canEditVideo` (prop) obscures the Skill Show tab intent.

Recommendation:
Rename the prop in `use-my-videos-actions.tsx`, `my-videos-table.tsx`, `my-videos-columns.tsx`, and `my-videos-actions-cell.tsx` from `canEditVideo` to `canDeleteDraft` (or `canDeleteDraftVideo`) for the delete-only contract. Optional follow-up — not a functional blocker if behavior is verified.

**PR comment (`my-videos-list-card.tsx` line 65):** **Medium:** `canDeleteDraft` is passed as `canEditVideo` — consider renaming the prop through the table/actions chain so Skill Show tab delete-only gating stays obvious and future edit actions aren't accidentally tied to this flag.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Library-upload retry skips thumbnail generation when `existingBackendId` is set | MEDIUM | Open | src/pages/videos/hooks/useVideoUpload.ts | 237-262, 408-431 |
| 2 | `canDeleteDraft` forwarded under misleading `canEditVideo` prop name | MEDIUM | Open | src/pages/videos/my-videos/dashboard/components/my-videos-list-card.tsx | 61-65, 131 |

**Merge readiness:** No Critical or High blockers. Two Medium items (library-upload retry thumbnails, prop naming) are acceptable to fix in follow-up or mark Accepted if retry-without-thumbnails is rare and prop rename is deferred.
