# PR review (SKSH-390) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#335](https://github.com/SkillshowFx/skillshow-admin-ui/pull/335) |
| Branch | `SKSH-390` → `main` |
| Head | `6fe2dab34eaf20c2fe58dd0ee5f0dfb13dda05e7` |
| Scope | Auto-expand uploaded video details when upload finishes; A–Z sport options in upload/edit flows |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-13 — head `6fe2dab` (gate auto-expand/select on empty expansion/selection) |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, audit-log UI stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Positive notes

- Upload-complete → expand uses a status-transition ref (`UPLOADING` → `READY`), so remounts/re-renders of already-ready rows do not force-open.
- Batch completions only auto-expand when `expandedId` is null (`current ?? firstReadyId`) and only auto-select when `selectedVideoId` is unset.
- Sport dropdowns in `VideoItem` / `EditVideoModal` reuse shared `sortSportSelectOptionsByLabel` (Other last).
- `accountService.getSportOptions` sorts at the API boundary for account/profile consumers.

## GitHub comments

_No open findings._

## Findings

---
Auto-expand steals focus during multi-file uploads

Risk Level: MEDIUM
File Path: src/pages/videos/components/UploadedVideosPanel.tsx
Lines: 58-71

Description:
**KISS** / UX — On every `UPLOADING` → `READY` transition the effect previously always `setExpandedId(video.id)` and called `onSelectedVideoChange`. During batch uploads, each completed file collapsed the previous row and changed selection, even if the user was already filling required fields on an earlier finished video.

Impact:
- Multi-file upload: later completions interrupted editing of an earlier ready video.
- Selection jumped to whichever file finished last in the effect loop.

Recommendation:
Only auto-expand when it will not interrupt in-progress editing, e.g. gate on no current expansion / selection, or expand only the first completion in a batch.

**Re-verify (6fe2dab):** ✅ Fixed — collects `newlyReadyIds`, uses only `firstReadyId`, `setExpandedId((current) => current ?? firstReadyId)`, and calls `onSelectedVideoChange` only when `!selectedVideoId`.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Auto-expand steals focus during multi-file uploads | MEDIUM | ✅ Fixed | `src/pages/videos/components/UploadedVideosPanel.tsx` | 58-71 |

**Merge readiness:** No open Critical/High/Medium blockers on frontend.
