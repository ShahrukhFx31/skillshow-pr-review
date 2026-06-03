# Frontend PR Review — skillshow-admin-ui (`SKSH-281`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-281`  
**Base:** `main...HEAD`  
**Initial review:** 2026-06-03  
**Re-reviewed:** 2026-06-03 — `e04059ca` (`fix: feedbacks`)  
**Scope:** Teams list server-side pagination/search, coach team select on upload/edit, team details overview actions, pagination bar layout (Critical, High & Medium)  
**Findings:** 2 Medium — **all resolved on branch**

---

## Overview

Teams list uses server-driven `getCoachTeams` with debounced search and `TeamTablePaginationBar`. Feedback commit adds `CoachTeamSelect` + `useCoachTeamSelectOptions` (server search, selected-team hydration) shared by upload and edit flows, and ties list loading skeletons to `pageSize`.

---

## Findings

---
Upload team picker capped at first 100 teams

Risk Level: MEDIUM  
File Path: src/pages/videos/hooks/useCoachTeamSelectOptions.ts (was `src/pages/videos/upload/index.tsx`)  
Lines: 16-43 (hook), `src/pages/videos/components/CoachTeamSelect.tsx`

Description:
Upload previously requested only `{ page: 1, pageSize: 100 }`, so coaches with more than 100 teams could not pick omitted teams.

Impact:
- Incomplete team dropdown on upload/edit for high-volume coaches.

Recommendation:
Paginate through all pages, add a lightweight options endpoint, or use server-side search so any team is discoverable.

**Re-review (`e04059ca`):** ✅ **Fixed** — Replaced inline fetch with `CoachTeamSelect` / `useCoachTeamSelectOptions`: debounced server `q` on `getCoachTeams`, `COACH_TEAM_SELECT_PAGE_SIZE` (10) with documented “search when you have more teams” intent, and `getCoachTeam(selectedId)` prepended when the current value is outside the result page. Upload (`VideoItem`) and `EditVideoModal` share this component.

---

---
Loading skeleton count fixed at six while page size defaults to five

Risk Level: MEDIUM  
File Path: src/pages/teams/index.tsx  
Lines: 127-136

Description:
Loading branch rendered six skeleton cards while default `pageSize` was 5.

Impact:
- Layout shift and mismatched loading affordance.

Recommendation:
Derive skeleton count from `pageSize`.

**Re-review (`e04059ca`):** ✅ **Fixed** — `Array.from({ length: pageSize }, () => undefined)` at line 129; skeleton keys use `TEAM_CARD_SKELETON_KEYS[idx] ?? \`teams-list-skel-${idx}\``.

---

## Positive notes

- **Server-driven list:** Debounced `q`, `keepPreviousData`, page reset on search, clamp when `total` shrinks.
- **Coach team select:** Reusable hook/component with `filterOption: false`, selected-team hydration, and visibility tied to unfiltered `total`.
- **Cache keys:** `coachTeamsAll()` invalidation still covers prefixed `coach-teams` queries.
- **Team details:** Overview actions in card `extra` via `onHeaderActionsChange`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Upload team picker capped at first 100 teams | MEDIUM | ✅ Fixed | src/pages/videos/hooks/useCoachTeamSelectOptions.ts | 16-43 |
| 2 | Loading skeleton count fixed at six while page size defaults to five | MEDIUM | ✅ Fixed | src/pages/teams/index.tsx | 127-136 |

**Merge readiness:** No open Critical/High/Medium blockers on `sksh-281` @ `e04059ca`.
