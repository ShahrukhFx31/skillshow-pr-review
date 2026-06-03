# Frontend PR Review — skillshow-admin-ui (`SKSH-297`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-297`  
**Base:** `main...HEAD`  
**Initial review:** 2026-06-03  
**Re-reviewed:** 2026-06-03 — `01e554e8` (`fix: update error handling and UI video editing modal render error and improve layout for thumbnail`)  
**Scope:** Video details/edit UX — team display, public toggle, sport/event fields, `EditVideoModal` expansion (Critical, High & Medium)  
**Findings:** 1 High, 2 Medium — **all resolved on branch**

---

## Overview

The PR expands video details with a `VideoDetailsFieldsView` grid (sport, event, team, visibility, metadata), enriches labels via `resolveCoachTeamLabel` + coach teams query, and replaces the narrow name/thumbnail edit flow with a full metadata modal. `BackendVideo` gains `teamId` / `teamName`; save uses `useUpdateVideoMutation`.

---

## Findings

---
Non-coach save re-sends `teamId` and triggers API 403

Risk Level: HIGH  
File Path: src/pages/videos/details/components/EditVideoModal.tsx  
Lines: 135

Description:
The PATCH body included `teamId` when `videoLevel.teamId` was set even if the viewer was not a coach, causing 403 from the API.

Impact:
- Save failed for non-coach editors on videos that already had `teamId`.

Recommendation:
Only include `teamId` when `isCoach` is true.

**Re-review (`01e554e8`):** ✅ **Fixed** — `...(isCoach ? { teamId: videoLevel.teamId ?? null } : {})` at line 135.

---

---
Edit modal requires sport and event even for legacy videos

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/EditVideoModal.tsx  
Lines: 103-151, 246-270

Description:
`handleSave` blocked submission unless `videoLevel.sportType` and `videoLevel.event` were set.

Impact:
- Legacy videos missing sport/event could not be updated from the details page.

Recommendation:
Align requiredness with backend optionality; send sport/event only when set.

**Re-review (`01e554e8`):** ✅ **Fixed** — Removed sport/event toasts and field errors; PATCH uses `...(videoLevel.event ? { eventType: videoLevel.event } : {})` and `...(videoLevel.sportType ? { sportType: videoLevel.sportType } : {})`; sport/event selects are optional with `allowClear`.

---

---
Custom thumbnail upload removed before frames/video URL are loaded

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/EditVideoModal.tsx  
Lines: 224-240

Description:
The pre-extraction branch only showed “Extract thumbnails from video” with no upload path.

Impact:
- Users could not upload a custom thumbnail without extracting frames first.

Recommendation:
Surface `ThumbnailPicker` upload control in the initial state.

**Re-review (`01e554e8`):** ✅ **Fixed** — `ThumbnailPicker` rendered in the `else` branch (line 239) without `videoUrl`, which exposes the built-in “Or upload custom thumbnail” file input when no frames are loaded.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Non-coach save re-sends `teamId` and triggers API 403 | HIGH | ✅ Fixed | src/pages/videos/details/components/EditVideoModal.tsx | 135 |
| 2 | Edit modal requires sport and event for legacy videos | MEDIUM | ✅ Fixed | src/pages/videos/details/components/EditVideoModal.tsx | 103-151, 246-270 |
| 3 | Custom thumbnail upload removed before frames loaded | MEDIUM | ✅ Fixed | src/pages/videos/details/components/EditVideoModal.tsx | 224-240 |

### Re-review notes (2026-06-03, `01e554e8`)

| Change | Verdict |
|--------|---------|
| `teamId` only when `isCoach` | ✅ Fixed (#1) |
| Optional sport/event; conditional PATCH fields | ✅ Fixed (#2) |
| `ThumbnailPicker` in pre-extraction branch | ✅ Fixed (#3) |
| `ThumbnailPicker.tsx` minor tweak (2-line) | No new blockers |

### Positive notes

- `VideoDetailsFieldsView` extracts detail layout from `VideoInfoCard`; team/visibility/share copy reuse `isVideoPublic`.
- Details page resolves `teamLabel` via API `teamName` with coach-team list fallback.
- Public visibility uses explicit `isPublic === true` consistently with upload flow and backend contract.

**Merge readiness:** No open Critical/High/Medium blockers on the frontend diff after `01e554e8`.
