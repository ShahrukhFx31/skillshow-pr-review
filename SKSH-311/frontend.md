# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full pr-review) — @ `b4957796`  
**Scope reviewed:** All **30 files** in `git diff main...HEAD` — My Videos dashboard refactor, server list hooks, linked athlete page, query param wiring.  
**Findings:** 1 prior High — **✅ Fixed**; **0 new** Critical/High/Medium

**Note:** Admin-table SKSH-311 archived in `Completed/SKSH-311/`. This report covers **My Videos server-side tabulation** (+ fix commit).

**Aligned with:** [backend.md](./backend.md)

### Full review coverage

| Area | Verdict |
|------|---------|
| `use-my-videos-dashboard-list.ts` — server table, meta, query keys | ✅ OK |
| `buildMyVideosListRequestParams` + linked athlete `includeSourceTab` | ✅ Fixed (#1) |
| `my-videos-table.tsx` / columns / actions / list card | ✅ OK |
| Linked athlete (`linked-athlete/index.tsx`) | ✅ OK |
| Removed `VideoTabsTable.tsx` — parity with server-side list | ✅ OK |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings added | 0 |
| Prior Open → Fixed | 1 |
| Remaining Open | 0 |

---

---
Linked athlete My Videos always sends `source=athlete` (hides Skill Show uploads)

Risk Level: HIGH  
File Path: src/pages/videos/my-videos/dashboard/utils.ts  
Lines: 56-65

Description (original):
Linked-athlete Media Vault always sent `source: athlete`, excluding `inVideoLibrary` videos.

**Re-review:** ✅ **Fixed** — `buildMyVideosListRequestParams` accepts `includeSourceTab?: boolean` (default true). `use-my-videos-dashboard-list.ts` sets `includeSourceTab = !isLinkedAthleteView` and omits `source` for linked athlete list + meta queries. Query keys use `sourceScopeKey` (`"all"` when tabs hidden).

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Linked athlete list always sends `source=athlete` | HIGH | ✅ Fixed | src/pages/videos/my-videos/dashboard/utils.ts | 56-65 |

## Positive notes

- Server-table pattern: `useServerTableControls`, `keepPreviousData`, meta query (`pageSize: 1`), `PaginationBar`.
- Filters (status, date, search, source tab) map to API params; sort keys `title` / `createdAt` match backend `sortBy`.
- Feature split into hooks + components matches admin-ui conventions.

**Merge readiness:** ✅ Clear — no Open findings. Backend partial filter fixed in [backend.md](./backend.md) #1.
