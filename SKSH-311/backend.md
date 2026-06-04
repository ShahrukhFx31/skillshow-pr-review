# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full pr-review) — @ `24ea8cd`  
**Scope reviewed:** All **3 files** in `git diff main...HEAD` — My Videos list query (`startDate`, `endDate`, `source`, `listStatus`) + shared `applyVideoCreatedAtRangeFilter`.  
**Findings:** **1 new High**, **0 Critical**

**Note:** Earlier SKSH-311 admin pagination work is merged to `main`; this pass covers the **My Videos server-side tabulation** commit only.

**Aligned with:** [frontend.md](./frontend.md)

### Full review coverage

| File | Reviewed |
|------|----------|
| `src/types/video.types.ts` | ✅ |
| `src/utils/video.utils.ts` | ✅ |
| `src/validation/video.validation.ts` | ✅ |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings added | 1 |
| Prior Open → Fixed | N/A (new slice) |
| Remaining Open | 1 |

---

---
`listStatus=partial` accepted by validation but not applied in list filter

Risk Level: HIGH  
File Path: src/utils/video.utils.ts  
Lines: 161-169

Description:
`videoListPaginatedQuerySchema` allows `listStatus: "partial"`, and the UI sends it for the Partial filter. `buildVideoDistributionListFilter` only maps `draft`, `processing`, `ready`, and `failed` to `processingStatus`. `partial` is a distribution aggregate (mixed completed/failed vendor logs per `computeVideoStatsFromDistributionDocs`), not a stored `processingStatus`, so the param is ignored and the API returns an unfiltered page.

Impact:
- Users selecting **Partial** on My Videos get wrong results and pagination totals after server-side migration.
- Filter contract disagrees between Joi, frontend, and Mongo filter logic.

Recommendation:
Implement a server-side partial filter consistent with stats logic (e.g. aggregation/`$expr` on vendor logs: at least one `completed` and one `failed`, excluding all-processing / all-failed / all-completed), or remove `"partial"` from Joi until supported and keep partial filtering client-side only.

```ts
} else if (listStatus === "partial") {
  // e.g. match docs where logs have both completed and failed vendors
}
```

**PR comment (line 161):** **High:** `listStatus=partial` is validated but never applied in `buildVideoDistributionListFilter` — Partial filter on My Videos will not work server-side. Please add partial matching (distribution logs) or drop `partial` from the schema until implemented.

**Status:** Open

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `listStatus=partial` validated but not filtered | HIGH | Open | src/utils/video.utils.ts | 161-169 |

## Positive notes

- `applyVideoCreatedAtRangeFilter` dedupes date-range logic for browser and distribution list filters.
- `source` athlete/skill maps cleanly to `inVideoLibrary` flags aligned with My Videos tabs.
- `startDate` / `endDate` wired through types, Joi, and filter builder consistently.

**Merge readiness:** Blocked on High #1 (`partial` list filter). See [frontend.md](./frontend.md) for linked-athlete `source` param.
