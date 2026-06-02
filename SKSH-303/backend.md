# Backend PR Review — skillshow (`SKSH-303`)

**Repo:** skillshow  
**Branch:** `sksh-303`  
**Base:** `main`  
**Re-reviewed:** 2026-06-01 — `origin/sksh-303` @ `064e25c` (`fix: feedback`)  
**Scope:** Coach dashboard API — remove `platform` query / `platformFilter`; all-vendor team leaderboard with batched DB access — Critical & High only  
**Findings:** 1 (0 Critical, 1 High) — **✅ Fixed**

**Aligned with:** [frontend.md](./frontend.md)

---

---
Sequential per-team DB work in `getDashboard`

Risk Level: HIGH  
File Path: src/services/coach-link.service.ts  
Lines: 373-497, 943-948

Description:
(Initial review.) `getDashboard` used a serial `await this.buildTeamRow(...)` per team (3–4 queries × team count), with all-vendor insights per call.

Impact:
- Dashboard latency scaled linearly with team count.

Recommendation:
(Initial.) Batch team metrics or parallelize; cap teams if needed.

**Re-review:** ✅ **Fixed** in `064e25c` — replaced with `buildTeamRowsForDashboard`, documented as fixed round-trips:

1. `findVideosByTeamsAndUsers(teamIds, orgUserIds)` — one query for all teams  
2. `findJobsByVideoIds(allVideoIds)` — one query  
3. `findInsightsByJobIds(jobIds)` — one query  
4. `findTopVideosLeanByIds(topVideoObjectIds)` — one batch for top clips  

In-memory maps (`videosByTeamId`, `viewsByTeamAndVideo`, `topPickByTeam`) build rows via `teamsRaw.map(...)` with no per-team awaits. Tests mock `findVideosByTeamsAndUsers` instead of per-team `findVideoIdsByTeamAndUsers`.

**PR comment** — anchor on a **changed** line in `fix: feedback` (`coach-link.service.ts` **line 373**, `buildTeamRowsForDashboard`):

> Addressed — batched team leaderboard removes per-team query loop. Thanks.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Sequential per-team DB work in `getDashboard` | HIGH | ✅ Fixed | src/services/coach-link.service.ts | 373-497, 943-948 |

## Positive notes

- **Batched access:** `findVideosByTeamsAndUsers` + single jobs/insights/top-video fetches; team count affects CPU mapping only, not query count.
- **Cross-platform totals:** Insights summed across `ALL_VENDORS` per video, rolled up per team.
- **API cleanup:** No `platform` query / `platformFilter`; swagger and controller tests updated.
- **Tests:** `coach-link.service.test.ts` updated for batched repository mocks and combined `totalViews` assertions.

**Note (not raised as High):** `buildPlatformViewsTotals` still runs a separate coach-wide insight pass; acceptable unless profiling shows duplicate load as a bottleneck.

**Merge readiness:** Backend clear — no open High/Critical on `origin/sksh-303` @ `064e25c`. With [frontend.md](./frontend.md) also clear, ticket **SKSH-303** is ready to archive under `pr-review/Completed/SKSH-303/` per README.
