# SKSH-303 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-303`  
**Base:** `main`  
**Re-verified:** 2026-06-02 — `origin/sksh-303` @ `5f64edfc` (no new commits since `fix: fe feedback`)  
**Scope:** Coach dashboard Teams card UX, Top Platforms progress chart, shared card layout constant, coach dashboard API without `platform` query param.

## Overview

The PR refocuses the coach analytics row: **Teams** drops the per-platform `Select`, improves the zero-teams empty state, and simplifies header actions. **Top Platforms** uses Ant `Progress` bars with `TOP_PLATFORMS_PROGRESS_MIN_SCALE`. Coach dashboard calls `getCoachDashboard()` without a `platform` param, aligned with batched all-vendor team metrics on the backend.

### Re-verification verdict (2026-06-02)

Re-checked `origin/sksh-303` @ `5f64edfc` against `origin/main`. **All four findings remain fixed.** No new Critical/High/Medium issues in the coach-dashboard diff.

| Check | Result |
|-------|--------|
| `getCoachDashboard()` without `platform` | ✅ |
| `isRefreshing` on `TopPlatformsChart` | ✅ |
| `CARD_HEAD_MOBILE_STACK_CLASS` in `src/constants/` | ✅ |
| `onCreateTeam` + empty-state CTA | ✅ |

---

## Findings

---
Team view counts still scoped to default platform without disclosure

Risk Level: MEDIUM  
File Path: src/pages/dashboard/coach/index.tsx  
Lines: 41-43

**Re-review:** ✅ **Fixed** — `queryFn: () => coachLinkService.getCoachDashboard()`; no platform state or query key segment.

---

---
Inconsistent loading UX between Teams and Top Platforms on refetch

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/TopPlatformsChart.tsx  
Lines: 35-36

**Re-review:** ✅ **Fixed** — `isLoading \|\| isRefreshing` skeleton; parent passes `isTeamsRefreshing`.

---

---
Dashboard imports team-feature layout constant

Risk Level: MEDIUM  
File Path: src/constants/card-layout.constants.ts

**Re-review:** ✅ **Fixed** — shared constant; teams re-export only.

---

---
Zero-teams empty state has no create-team action

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/TopPerformingTeamsCard.tsx  
Lines: 38-40

**Re-review:** ✅ **Fixed** — `onCreateTeam` prop + primary button in `Empty`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Team view counts still scoped to default platform without disclosure | MEDIUM | ✅ Fixed | src/pages/dashboard/coach/index.tsx | 41-43 |
| 2 | Inconsistent loading UX between Teams and Top Platforms on refetch | MEDIUM | ✅ Fixed | src/pages/dashboard/components/TopPlatformsChart.tsx | 35-36 |
| 3 | Dashboard imports team-feature layout constant | MEDIUM | ✅ Fixed | src/constants/card-layout.constants.ts | — |
| 4 | Zero-teams empty state has no create-team action | MEDIUM | ✅ Fixed | src/pages/dashboard/components/TopPerformingTeamsCard.tsx | 38-40 |

## Positive notes

- **`TOP_PLATFORMS_PROGRESS_MIN_SCALE`:** Sensible bar scaling for low view counts.
- **Refetch UX:** Shared `isTeamsRefreshing` / `isRefreshing` on both analytics cards.
- **`card-layout.constants.ts`:** Shared layout utility in the right layer.
- **Empty state:** Icon, copy, and create CTA for zero teams.

**Backend:** [backend.md](./backend.md) — `buildTeamRowsForDashboard` @ `064e25c`.

**Merge readiness:** Frontend clear — no open findings on `origin/sksh-303` @ `5f64edfc`.
