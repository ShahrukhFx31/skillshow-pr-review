# Frontend PR Review — skillshow-admin-ui (`SKSH-281`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-281`  
**Base:** `main...HEAD`  
**Commits:** `94b077bb`, `78c9ea7b`, `081e7dd5`, `b65a502c` (QA), merge from `main`  
**Scope:** Teams list server-side pagination/search, upload team picker API shape, team details overview actions in card header, pagination bar responsive layout (Critical, High & Medium)

---

## Overview

Teams list drops client-side filtering in favor of `getCoachTeams({ page, pageSize, q })` with debounced search, `keepPreviousData`, and `TeamTablePaginationBar`. Query keys split `coachTeams(search, page, pageSize)` and `coachTeamsAll()` for invalidation. Team details lifts edit/save actions into the card `extra` via `onHeaderActionsChange`. Upload video page adapts to `CoachTeamsPageDto` with a single `pageSize: 100` fetch.

---

---
Upload team picker capped at first 100 teams

Risk Level: MEDIUM  
File Path: src/pages/videos/upload/index.tsx  
Lines: 96-104

Description:
After pagination, `getCoachTeams` returns at most 100 items per request. The upload flow requests only `{ page: 1, pageSize: 100 }` and builds `coachTeamOptions` from that page. Coaches with more than 100 teams will not see newer teams in the dropdown (sorted by `createdAt` desc on the API).

Impact:
- Upload (and any flow using this query) can omit valid teams from the selector with no user-visible warning.
- Regression vs the previous unbounded list for high-volume coaches.

Recommendation:
Either fetch all pages until `items.length >= total` (with a sane upper bound), add a dedicated “team options” endpoint, or document/enforce a product max and surface a message when `total > items.length`. Example loop:

```typescript
const fetchAllCoachTeams = async () => {
  const pageSize = 100;
  let page = 1;
  let items: CoachTeamListItemDto[] = [];
  let total = 0;
  do {
    const res = await coachLinkService.getCoachTeams({ page, pageSize });
    items = items.concat(res.items);
    total = res.total;
    page += 1;
  } while (items.length < total && page <= Math.ceil(total / pageSize));
  return items;
};
```

**PR comment (line 98):** Upload now only loads the first page (`pageSize: 100`) of coach teams. Coaches with >100 teams will miss teams in the upload dropdown. Can we paginate through all pages (or add a lightweight options endpoint) so the selector stays complete?

---

---
Loading skeleton count fixed at six while page size defaults to five

Risk Level: MEDIUM  
File Path: src/pages/teams/index.tsx  
Lines: 127-137

Description:
The loading branch always renders six skeleton cards (`Array.from({ length: 6 })`) while list pagination defaults to `DEFAULT_LIST_PAGINATION.pageSize` (5 from `PAGE_SIZE_OPTIONS[0]`). When `pageSize` is 5 (or the user selects another size), the skeleton grid no longer matches the eventual row count.

Impact:
- Brief layout shift and mismatched loading affordance on the teams grid.
- Harder to scan loading state on non-default page sizes.

Recommendation:
Derive skeleton count from `pageSize` (or `Math.min(pageSize, 6)` if you want a cap on large sizes):

```typescript
Array.from({ length: pageSize }).fill(undefined)
```

**PR comment (line 129):** Skeleton placeholders are hardcoded to 6 but default page size is 5. Consider tying skeleton count to `pageSize` so loading matches the paginated grid.

---

## Positive notes

- **Server-driven list:** Debounced `q`, `keepPreviousData`, page reset on search, and clamp when `total` shrinks — solid pagination UX.
- **Cache keys:** `coachTeamsAll()` prefix invalidation covers list + upload `["coach-teams", "all-options"]` after team updates.
- **Team details:** Overview edit/save/cancel moved to card `extra` with `useLayoutEffect` cleanup — avoids duplicate controls in the tab body.
- **`TeamTablePaginationBar`:** ResizeObserver-based stack-on-overflow improves narrow viewports without always forcing column layout.
- **Types/API:** `CoachTeamsPageDto` / `CoachTeamsParams` align with backend paginated contract.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Upload team picker capped at first 100 teams | MEDIUM | Open | src/pages/videos/upload/index.tsx | 96-104 |
| 2 | Loading skeleton count fixed at six while page size defaults to five | MEDIUM | Open | src/pages/teams/index.tsx | 127-137 |

**Merge readiness:** No Critical/High blockers. Two Medium items: upload dropdown completeness for >100 teams, and skeleton count vs `pageSize`.
