# Frontend PR Review — skillshow-admin-ui (`SKSH-297`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-297`  
**Base:** `main...HEAD`  
**Scope:** Video details/edit UX — team display, public toggle, sport/event fields, `EditVideoModal` expansion (Critical, High & Medium)  
**Findings:** 1 High, 2 Medium

---

## Overview

The PR expands video details with a `VideoDetailsFieldsView` grid (sport, event, team, visibility, metadata), enriches labels via `resolveCoachTeamLabel` + coach teams query, and replaces the narrow name/thumbnail edit flow with a full metadata modal (sport, event, team, public switch, share URL copy). `BackendVideo` gains `teamId` / `teamName`; save uses `useUpdateVideoMutation` instead of `useEditVideoDetailsMutation`.

---

---
Non-coach save re-sends `teamId` and triggers API 403

Risk Level: HIGH  
File Path: src/pages/videos/details/components/EditVideoModal.tsx  
Lines: 143-147

Description:
The PATCH body includes `teamId` when `videoLevel.teamId` is set even if the viewer is not a coach:

```typescript
...(isCoach && coachTeamOptions.length > 0
  ? { teamId: videoLevel.teamId ?? null }
  : videoLevel.teamId
    ? { teamId: videoLevel.teamId }
    : {}),
```

The backend now rejects non-coach `teamId` mutations with 403 (`Coach role required`). Any save that echoes an existing `teamId` without coach role fails even when the user only changes name, sport, or visibility.

Impact:
- Save fails with 403 for non-coach editors on videos that already have `teamId` (e.g. parent/coach linked-athlete flows, multi-role edge cases, or stale role state).
- User sees a failed save with no path to update other fields unless `teamId` is omitted from the payload.

Recommendation:
Only include `teamId` when `isCoach` is true. Omit the field entirely for non-coaches so PATCH updates other metadata without touching team assignment:

```typescript
...(isCoach ? { teamId: videoLevel.teamId ?? null } : {}),
```

**PR comment (line 143):** **High:** Non-coaches still send `teamId` when the video already has one, but the API now returns 403 for non-coach team patches. Please include `teamId` in the PATCH body only when `isCoach` is true; omit it otherwise so name/sport/visibility edits still save.

---

---
Edit modal requires sport and event even for legacy videos

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/EditVideoModal.tsx  
Lines: 115-121, 167-168

Description:
`handleSave` blocks submission unless `videoLevel.sportType` and `videoLevel.event` are set, with client-side toasts/errors. Previously the edit modal only required a non-empty name.

Impact:
- Videos uploaded before sport/event were required cannot be updated from the details page (e.g. rename or thumbnail only).
- Users with incomplete metadata hit validation errors with no way to save other changes from this modal.

Recommendation:
Align requiredness with backend Joi (optional fields) or pre-fill from existing `video.sportType` / `video.eventType` and allow save when unchanged. If product requires sport/event going forward, migrate existing rows or show a dedicated onboarding path instead of hard-blocking all edits.

**PR comment (line 115):** **Medium:** Requiring sport and event on every edit blocks updating legacy videos that never had those fields. Consider matching backend optionality or gating the requirement only when those fields are empty and the user is distributing.

---

---
Custom thumbnail upload removed before frames/video URL are loaded

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/EditVideoModal.tsx  
Lines: main 168–181 (removed); PR +253

Description:
The previous modal offered a file input to upload a custom thumbnail when backend frames were unavailable (`Or upload a custom image below` + `<input type="file" …>` on `main`, lines 168–181). This PR deletes that block and closes the branch with `)}` at PR line 253. The “Extract thumbnails from video” button text (PR line 251) is unchanged diff context — not a changed line in GitHub.

Impact:
- Users who relied on direct image upload without extracting frames first lose that workflow in the edit modal.
- Extra step (network play-url fetch + frame extraction) required before custom thumbnail selection.

Recommendation:
Restore a file input in the pre-extraction branch (as before), or render `ThumbnailPicker` without `videoUrl` so its built-in upload control remains available.

**PR comment (deleted line 168 on base, left diff pane):** **Medium:** This PR removes the “Or upload a custom image below” file input that used to sit under the extract button. Please restore direct upload or surface `ThumbnailPicker`’s upload control before frames/video URL are loaded.

*(Alternate anchor on PR branch: line 253 `)}` — the addition that replaces the removed upload block.)*

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Non-coach save re-sends `teamId` and triggers API 403 | HIGH | Open | src/pages/videos/details/components/EditVideoModal.tsx | 143-147 |
| 2 | Edit modal requires sport and event for legacy videos | MEDIUM | Open | src/pages/videos/details/components/EditVideoModal.tsx | 115-121, 167-168 |
| 3 | Custom thumbnail upload removed before frames loaded | MEDIUM | Open | src/pages/videos/details/components/EditVideoModal.tsx | main 168–181 (removed); PR +253 |

### Positive notes

- `VideoDetailsFieldsView` extracts detail layout from `VideoInfoCard`; team/visibility/share copy are composable and reuse `isVideoPublic`.
- Details page resolves `teamLabel` via API `teamName` with coach-team list fallback (`resolveCoachTeamLabel`).
- Public visibility uses explicit `isPublic === true` consistently with upload flow and backend contract.
- `mapVideoInfo` and types extended cleanly; coach teams query is scoped with `enabled: isCoach`.

**Merge readiness:** Blocked on High #1 (403 on non-coach save when `teamId` is present). Medium items are UX/regression follow-ups.
