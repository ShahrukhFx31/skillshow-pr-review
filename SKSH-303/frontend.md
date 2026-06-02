# SKSH-303 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-303`  
**Base:** `main`  
**Re-reviewed:** 2026-06-01 — `origin/sksh-303` @ `5f64edfc` (`fix: fe feedback`)  
**Scope:** Coach dashboard Teams card UX, Top Platforms progress chart, shared card layout constant, coach dashboard API without `platform` query param.

## Overview

The PR refocuses the coach analytics row: **Teams** drops the per-platform `Select`, improves the zero-teams empty state, and simplifies header actions. **Top Platforms** uses Ant `Progress` bars with `TOP_PLATFORMS_PROGRESS_MIN_SCALE`. Coach dashboard calls `getCoachDashboard()` without a `platform` param, aligned with batched all-vendor team metrics on the backend.

### Re-review verdict (latest)

After `fix: fe feedback` (`5f64edfc`): **all findings resolved.** Empty-state create CTA restored (`onCreateTeam` + primary button in `Empty`).

---

## Findings

---
Team view counts still scoped to default platform without disclosure

Risk Level: MEDIUM  
File Path: src/pages/dashboard/coach/index.tsx  
Lines: 41-43

Description:
(Initial review.) Platform picker removed while API still used Instagram-only `buildTeamRowForPlatform`.

**Re-review:** ✅ **Fixed** — `getCoachDashboard()` without `platform`; backend `buildTeamRowsForDashboard` aggregates views across all vendors.

---

---
Inconsistent loading UX between Teams and Top Platforms on refetch

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/TopPlatformsChart.tsx  
Lines: 35-36

Description:
(Initial review.) Teams skeletoned on `isFetching`; Top Platforms only on initial load.

**Re-review:** ✅ **Fixed** — `isRefreshing` + `isLoading || isRefreshing` skeleton.

---

---
Dashboard imports team-feature layout constant

Risk Level: MEDIUM  
File Path: src/constants/card-layout.constants.ts

Description:
(Initial review.) Dashboard imported `TEAM_CARD_HEAD_MOBILE_STACK_CLASS` from teams feature.

**Re-review:** ✅ **Fixed** — Shared `CARD_HEAD_MOBILE_STACK_CLASS` in `src/constants/`; teams re-export alias.

---

---
Zero-teams empty state has no create-team action

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/TopPerformingTeamsCard.tsx  
Lines: 38-40  
File Path: src/pages/dashboard/coach/index.tsx  
Lines: 197-198

Description:
(Initial re-review.) `onCreateTeam` and empty-state CTA were removed in an earlier feedback pass.

**Re-review:** ✅ **Fixed** in `5f64edfc` — `onCreateTeam` wired from coach page; `Empty` footer restored:

```tsx
<Button onClick={onCreateTeam} type="primary">
  <Icon icon="ic:baseline-plus" size={18} /> Create Your First Team
</Button>
```

**PR comment (`TopPerformingTeamsCard.tsx` line 38 — changed in `fix: fe feedback`):**  
Addressed — create-team CTA restored in empty state.

---

## Positive notes

- **`TOP_PLATFORMS_PROGRESS_MIN_SCALE`:** Sensible bar scaling for low view counts.
- **Refetch UX:** Shared `isTeamsRefreshing` / `isRefreshing` on both analytics cards.
- **`card-layout.constants.ts`:** Shared layout utility in the right layer.
- **Empty state:** Icon, copy, and create CTA for zero teams.

**Backend:** See [backend.md](./backend.md) — batched `buildTeamRowsForDashboard` in `064e25c`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Team view counts still scoped to default platform without disclosure | MEDIUM | ✅ Fixed | src/pages/dashboard/coach/index.tsx | 41-43 |
| 2 | Inconsistent loading UX between Teams and Top Platforms on refetch | MEDIUM | ✅ Fixed | src/pages/dashboard/components/TopPlatformsChart.tsx | 35-36 |
| 3 | Dashboard imports team-feature layout constant | MEDIUM | ✅ Fixed | src/constants/card-layout.constants.ts | — |
| 4 | Zero-teams empty state has no create-team action | MEDIUM | ✅ Fixed | src/pages/dashboard/components/TopPerformingTeamsCard.tsx | 38-40 |

**Merge readiness:** Frontend clear — no open findings on `origin/sksh-303` @ `5f64edfc`.
