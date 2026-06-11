# Backend PR Review — skillshow (`SKSH-314`)

**Repo:** skillshow  
**Branch:** `SKSH-314`  
**Base:** `main...HEAD` @ `8ce66d29`  
**Reviewed:** 2026-06-11  
**Scope:** Server-driven app-user **Linked Events** list (`GET /v1/app-users/:appUserId/linked-events`); remove embedded `linkedEvents` from detail payload; source events from athlete registrations instead of video `eventId` distinct  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 9 — `event-athlete.repository`, `app-user.service`, controller/routes/validation/types/constants, remove `video.repository.findDistinctEventIdsByUserLean`, controller test

**Findings:** 1 Open (0 Critical, 1 High)

### Protected modules

No changes to frozen `list-query.validation.ts`, `list-query-aggregation.utils.ts`, change-stream, or audit-log modules. New list route correctly uses `createListQuerySchema` + `validatedQuery` + shared sort/pagination constants.

### Positive notes

- Controller follows established `listTeams` / `listActivityVideos` pattern (`DEFAULT_LIST_QUERY` merge, thin handler).
- Validation uses `createListQuerySchema` with `APP_USER_LINKED_EVENTS_LIST_SORT_BY_VALUES`; sort field map in `app-user.constants.ts`.
- Search uses `escapeRegex`; soft-deleted events filtered via `"event.isDeleted": { $ne: true }`.
- Non-athletes return empty paginated page; frontend hides tab — aligned.
- `linkedEvents` removed from `AppUserDetail` in sync with new paginated endpoint.

---

## GitHub comments (Open findings)

### 1. `src/repositories/event-athlete.repository.ts` line 156

`listLinkedEventsPaginatedByAthlete` hand-rolls a `$facet` pagination pipeline even though this file already imports `runListQueryAggregate` and uses it in `listRowsForEvent` (line 225). Refactor to `runListQueryAggregate` / `runPaginatedAggregate` with `stagesBeforeMatch` (athlete match + event `$lookup` + `$unwind`) and `match` (`event.isDeleted` + `buildRegexOrMatch` on `event.eventName` / `event.city` / `event.state`). That keeps pagination, sort tie-break (`_id: -1`), and count strategy consistent with every other list repo in this codebase.

---

---
Ad-hoc `$facet` duplicates `runListQueryAggregate`

Risk Level: HIGH  
File Path: src/repositories/event-athlete.repository.ts  
Lines: 143-188

Description:
**DRY / Contract.** `listLinkedEventsPaginatedByAthlete` builds a manual `$facet` (`items` + `total` branches with `$sort` / `$skip` / `$limit` / `$count`) while the same repository already calls `runListQueryAggregate` for `listRowsForEvent`. The file imports both `buildRegexOrMatch` and `runListQueryAggregate` but the new method inlines regex search with a separate `escapeRegex` block and a parallel facet implementation.

Impact:
- Pagination/sort/count behavior can drift from other list endpoints (e.g. missing `_id` tie-break guarantees, different count strategy).
- Future changes to `list-query-aggregation.utils` will not apply to linked events without a second edit path.
- Maintainability debt in the one repo that mixes both patterns.

Recommendation:
Refactor `listLinkedEventsPaginatedByAthlete` to delegate to `runPaginatedAggregate` or `runListQueryAggregate`:

```typescript
const searchMatch = buildRegexOrMatch(opts.search, [
  "event.eventName",
  "event.city",
  "event.state",
]);

return runPaginatedAggregate({
  model: EventAthleteRegistrationModel,
  stagesBeforeMatch: this.buildAthleteLinkedEventsLookupStages(athleteId, { search: undefined }),
  match: {
    "event.isDeleted": { $ne: true },
    ...searchMatch,
  },
  sort: opts.sort,
  page: /* derive from skip/limit or pass query */,
  pageSize: opts.limit,
  project: {
    $project: {
      _id: "$event._id",
      city: "$event.city",
      eventName: "$event.eventName",
      slug: "$event.slug",
      startDate: "$event.startDate",
      state: "$event.state",
    },
  },
});
```

Alternatively pass the full `ListQuery` into the repository and use `runListQueryAggregate` with `APP_USER_LINKED_EVENTS_LIST_SORT_FIELD_MAP` (import from `app-user.constants.ts` or accept `sortFieldMap` as a parameter). Drop the inline `$facet` block entirely.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Ad-hoc `$facet` duplicates `runListQueryAggregate` | HIGH | Open | src/repositories/event-athlete.repository.ts | 143-188 |

**Merge readiness:** **Request changes** — 1 open High (DRY / list aggregation contract). No Critical findings; list validation and controller wiring are otherwise sound.
