# PR review (SKSH-390) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#335](https://github.com/SkillshowFx/skillshow-admin-ui/pull/335) |
| Branch | `SKSH-390` → `main` |
| Head | `630465145207da2fac9a34167c27b9f64ed9fe25` |
| Scope | Auto-expand uploaded video details when upload finishes; A–Z sport options in upload/edit flows |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, audit-log UI stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Positive notes

- Upload-complete → expand uses a status-transition ref (`UPLOADING` → `READY`), so remounts/re-renders of already-ready rows do not force-open.
- Sport dropdowns in `VideoItem` / `EditVideoModal` reuse shared `sortSportSelectOptionsByLabel` (Other last).
- `accountService.getSportOptions` sorts at the API boundary for account/profile consumers.

## GitHub comments

### `src/pages/videos/components/UploadedVideosPanel.tsx`
- **MEDIUM** — Auto-expand steals focus during multi-file uploads (line 67)

## Findings

---
Auto-expand steals focus during multi-file uploads

Risk Level: MEDIUM
File Path: src/pages/videos/components/UploadedVideosPanel.tsx
Lines: 58-71

Description:
**KISS** / UX — On every `UPLOADING` → `READY` transition the effect always `setExpandedId(video.id)` and calls `onSelectedVideoChange`. During batch uploads, each completed file collapses the previous row and changes selection, even if the user is already filling required fields on an earlier finished video.

Impact:
- Multi-file upload: later completions interrupt editing of an earlier ready video.
- Selection jumps to whichever file finished last in the effect loop.

Recommendation:
Only auto-expand when it will not interrupt in-progress editing, e.g. gate on no current expansion / selection:

```typescript
if (justFinishedUpload) {
  setExpandedId((current) => current ?? video.id);
  if (!selectedVideoId) {
    onSelectedVideoChange?.(video.id);
  }
}
```

Or expand only the first completion in a batch and leave subsequent finishes collapsed until the user opens them.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Auto-expand steals focus during multi-file uploads | MEDIUM | Open | `src/pages/videos/components/UploadedVideosPanel.tsx` | 58-71 |

**Merge readiness:** No Critical/High blockers. One Medium UX follow-up on multi-file auto-expand.
