# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full pr-review) — @ `a80e6bb`  
**Scope reviewed:** All **19 files** in `git diff main...HEAD` — events server list + My Videos list filters/distribution pipeline, tests.  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency enforced)

**Aligned with:** [frontend.md](./frontend.md)

### Full review coverage

| File group | Reviewed |
|------------|----------|
| `event.*` (controller, service, repository, validation, constants, types, tests) | ✅ |
| `video.*` (utils, pipeline, list-status, repository, service, validation, types, tests) | ✅ |
| `vendor.constants.ts` | ✅ |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings added | 0 |
| Prior Open → Fixed this pass | 1 |
| Remaining Open | 0 |

---

## GitHub comments (Open findings)

None.

---

---
`listStatus=partial` accepted by validation but not applied in list filter

Risk Level: HIGH  
File Path: src/utils/video-distribution-pipeline.utils.ts  
Lines: 53-58

Description:
`listStatus=partial` was validated but ignored in the pre-aggregation Mongo match.

Impact:
- Partial filter on My Videos returned unfiltered pages after server-side migration.

Recommendation:
Use distribution pipeline + `distributionListStatus` classifier (implemented).

**Re-review:** ✅ **Fixed** — `countWithDistributionFilter` / `aggregateWithDistribution` apply `buildDistributionListStatusAddFieldsStage()` and `$match: { distributionListStatus: listStatus }`. Classifier shared with stats; tests in `video.utils.test.ts`.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Event `getEvents` merges raw `req.query` (Global consistency)

Risk Level: HIGH  
File Path: src/controllers/event.controller.ts  
Lines: 44-47

Description (original):
`getEvents` used `(validatedQuery ?? req.query)`, unlike other list controllers migrated to `validatedQuery` + `DEFAULT_LIST_QUERY` only.

Impact:
- Raw query strings could bypass Joi coercion for `pageSize`, `sortBy`, `status`.
- **Global consistency** violation within the same PR’s list-endpoint pattern.

Recommendation:
```ts
const query = {
  ...DEFAULT_LIST_QUERY,
  ...(req as ValidatedQueryRequest).validatedQuery,
} as EventListQuery;
```

**Re-review:** ✅ **Fixed** at lines 44–47. Controller tests updated to pass `validatedQuery` for filter/pagination cases.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `listStatus=partial` not filtered server-side | HIGH | ✅ Fixed | src/utils/video-distribution-pipeline.utils.ts | 53-58 |
| 2 | Event `getEvents` merges raw `req.query` fallback | HIGH | ✅ Fixed | src/controllers/event.controller.ts | 44-47 |

## Positive notes (DRY / KISS / Global consistency)

- **DRY:** Events use shared `createListQuerySchema`, `EVENT_LIST_SORT_FIELD_MAP`, `runListQueryAggregate` (replaces bespoke `listPageWithTotal` `$facet`). Video list status centralized in `video-distribution-list-status.utils.ts`.
- **Global consistency:** Event list aligns with app-user/partner/video-library pattern (`validatedQuery`, `pageSize`, `status` query param).
- **KISS:** Event service delegates pagination to repository `listRows`; video `listStatus` filter uses one pipeline builder.

**Merge readiness:** ✅ Clear — **0 Open** Critical/High. Aligned with [frontend.md](./frontend.md).
