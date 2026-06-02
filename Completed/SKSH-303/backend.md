# Backend PR Review — skillshow (`SKSH-303`)

**Repo:** skillshow  
**Branch:** `sksh-303`  
**Base:** `main`  
**Re-verified:** 2026-06-02 — `origin/sksh-303` @ `064e25c` (no new commits since `fix: feedback`)  
**Scope:** Coach dashboard API — remove `platform` query / `platformFilter`; all-vendor team leaderboard with batched DB access — Critical & High only  
**Findings:** 1 (0 Critical, 1 High) — **✅ Fixed**

**Aligned with:** [frontend.md](./frontend.md)

### Re-verification verdict (2026-06-02)

Re-checked `origin/sksh-303` @ `064e25c` against `origin/main`. **Finding #1 remains fixed.** No new Critical/High issues in the coach-dashboard diff.

| Check | Result |
|-------|--------|
| `getDashboard(coachUserId)` only — no `platform` query | ✅ |
| `buildTeamRowsForDashboard` (batched queries) | ✅ |
| No `buildTeamRowForPlatform` / serial per-team loop | ✅ |
| `platformFilter` removed from DTO | ✅ |

---

---
Sequential per-team DB work in `getDashboard`

Risk Level: HIGH  
File Path: src/services/coach-link.service.ts  
Lines: 373-497, 943-948

Description:
(Initial review.) Serial `await this.buildTeamRow(...)` per team (3–4 queries × team count).

**Re-review:** ✅ **Fixed** in `064e25c` — `buildTeamRowsForDashboard` uses four batched repository calls (`findVideosByTeamsAndUsers`, `findJobsByVideoIds`, `findInsightsByJobIds`, `findTopVideosLeanByIds`) then maps rows in memory. `getDashboard` calls it once (line 943).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Sequential per-team DB work in `getDashboard` | HIGH | ✅ Fixed | src/services/coach-link.service.ts | 373-497, 943-948 |

## Positive notes

- **Batched access:** Fixed DB round-trips regardless of team count.
- **Cross-platform totals:** Insights summed across `ALL_VENDORS` per video, per team.
- **API cleanup:** Controller, swagger, and types aligned with no `platform` filter.
- **Tests:** `coach-link.service.test.ts` uses `findVideosByTeamsAndUsers` mocks and cross-vendor `totalViews` assertions.

**Note (not raised as High):** `buildPlatformViewsTotals` still runs a separate coach-wide insight pass — acceptable unless profiling shows it as a bottleneck.

**Merge readiness:** Backend clear — no open High/Critical on `origin/sksh-303` @ `064e25c`. With [frontend.md](./frontend.md), ticket **SKSH-303** is ready to move to `pr-review/Completed/SKSH-303/` per README.
