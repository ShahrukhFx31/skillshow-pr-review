# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full pr-review) — @ `e9c69f5b`  
**Scope reviewed:** All **30 files** in `git diff main...HEAD` — My Videos refactor (`my-videos/dashboard/*`), list query wiring, router/breadcrumb moves, `useVideoListFilters` / `use-pagination` touch.  
**Findings:** **1 new High**, **0 Critical**, **0 new Medium**

**Note:** Prior SKSH-311 admin-table review is archived under `Completed/SKSH-311/`. This pass is the **My Videos server-side tabulation** slice on the same branch.

**Aligned with:** [backend.md](./backend.md)

### Full review coverage

| Area | Verdict |
|------|---------|
| `use-my-videos-dashboard-list.ts` — server table, meta query, query keys | ✅ OK (see High #1 for linked athlete `source`) |
| `buildMyVideosListRequestParams` / filter controls | ⚠️ High #1 |
| `my-videos-table.tsx` — server sort, bulk N/A, error/empty states | ✅ OK |
| `use-my-videos-actions.tsx` — delete/retry/navigation | ✅ OK |
| Linked athlete page (`linked-athlete/index.tsx`) | ⚠️ High #1 |
| Removed `VideoTabsTable.tsx` — behavior parity checked vs `main` | ✅ (with #1 regression) |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings added | 1 |
| Prior Open → Fixed | N/A (new slice) |
| Remaining Open | 1 |

---

---
Linked athlete My Videos always sends `source=athlete` (hides Skill Show uploads)

Risk Level: HIGH  
File Path: src/pages/videos/my-videos/dashboard/utils.ts  
Lines: 49-72

Description:
`buildMyVideosListRequestParams` always sets `source: resolveMyVideosUploadSource(args.sourceTab)` (defaults to athlete). Linked-athlete / Media Vault uses `hideSourceTabs: true` but still passes `sourceTab: "athlete"`. On `main`, the list fetched all videos for the relation (no `source` query) and did not server-filter by `inVideoLibrary`. After migration, `source=athlete` adds `inVideoLibrary: { $ne: true }`, excluding videos assigned via Video Library (`inVideoLibrary: true`) from the athlete’s vault.

Impact:
- Parents/coaches viewing a linked athlete’s Media Vault may not see Skill Show–assigned videos that appeared before this change.
- Incorrect empty state and totals for linked profiles with library-sourced videos.

Recommendation:
When `mode === "linkedAthlete"` or `hideSourceTabs`, omit `source` from list params (and meta query) so the API returns the full scoped set for that relation:

```ts
...(includeSourceTab ? { source: resolveMyVideosUploadSource(args.sourceTab) } : {}),
```

**PR comment (line 64):** **High:** Linked athlete Media Vault still sends `source: athlete` when source tabs are hidden — this filters out `inVideoLibrary` videos. Omit `source` for linked-athlete list/meta requests (same as pre-migration “all videos for relation” behavior).

**Status:** Open

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Linked athlete list always sends `source=athlete` | HIGH | Open | src/pages/videos/my-videos/dashboard/utils.ts | 49-72 |

## Positive notes

- My Videos dashboard follows established server-table pattern (`useServerTableControls`, `keepPreviousData`, meta query with `pageSize: 1`, `PaginationBar`).
- Status/date/search filters passed to API via `buildMyVideosListRequestParams`; no client-side pagination of full catalog.
- Column sort keys (`title`, `createdAt`) match backend `sortBy` values.
- Feature split into hooks/components matches project layout conventions.

**Merge readiness:** Blocked on High #1 and [backend.md](./backend.md) High #1 (`partial` filter). Partial filter also affects owner My Videos when user selects Partial in UI.
