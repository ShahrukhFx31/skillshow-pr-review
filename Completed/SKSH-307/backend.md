# Backend PR Review — skillshow (`SKSH-307`)

**Repo:** skillshow  
**Branch:** `sksh-307`  
**Base:** `main...HEAD`  
**Re-review:** `9f58663` (fix: fix public), `00394c3` (fix: public only issue); **post-merge:** `e7d4f5d` (merge `main` into `sksh-307`)  
**Scope:** Paginated linked-athlete Media Pool APIs (`GET /v1/coach/media-pool`, `GET /v1/parent-link/media-pool`), `relationId` on parent dashboard athlete feed, shared video repository queries (Critical & High only)

---

## Overview

Adds `mediaPool` handlers on coach and parent link controllers, routes, and services. Coach `getAthleteMediaPool` mirrors parent scope building (accepted relations + `linkedAt`), paginates via `videoRepository.countLinkedAthletesVideosLean` / `findLinkedAthletesFeedPaginatedLean`, and returns `MediaPoolFeedPage` with presigned thumbnails and `relationId`. Parent service refactors scope/name helpers and reuses the same repository methods; dashboard `athleteFeed` now includes `relationId` and drops videos without a resolvable relation.

**Re-review:** Linked-athlete video queries now apply `VIDEO_PUBLIC_ONLY_QUERY` (`isPublic: true`) via shared `linkedAthletesVideoFilter`, including `findRecentByLinkedAthletesLean` (parent dashboard athlete feed) and new Media Pool pagination paths — aligns Media Pool with Media Vault visibility rules.

**Post-merge re-review (`e7d4f5d`):** No conflict markers. Media-pool routes/controllers/services, `linkedAthletesVideoFilter` + `VIDEO_PUBLIC_ONLY_QUERY`, and parent dashboard `relationId` feed mapping intact after merge with `main`. Unrelated formatting-only diff (`list-query-aggregation.utils.ts`) no longer in branch diff. No new Critical/High findings.

---

## Findings

No **Critical** or **High** issues identified in scope.

---

## Positive notes

- **Layering:** Controllers stay thin (auth/role gate on coach, parse query, delegate); Mongo access in `videoRepository`; mapping in services.
- **Pagination:** `pageSize` capped at 50 in both util and service; empty scopes return stable `{ items: [], total: 0 }`.
- **Reads:** `.lean()` + `DASHBOARD_VIDEO_SELECT` on list path; shared `linkedAthletesVideoFilter` for count, page, and recent linked-athlete feeds.
- **Visibility:** `VIDEO_PUBLIC_ONLY_QUERY` applied consistently on all linked-athlete list surfaces (vault, media pool, dashboard athlete feed).
- **Auth:** Coach endpoint checks coach role before service call; routes sit under global `/api/v1` auth.
- **Types:** `MediaPoolFeedItem`, `MediaPoolFeedPage`, `MediaPoolListOpts` in `src/types/media-pool.types.ts`.
- **Tests:** Parent dashboard test asserts `relationId` on athlete feed items.

**Note (not raised as High):** Media-pool routes use `parseMediaPoolQuery(req.query)` like existing coach list endpoints (`parseCoachAthleteListQuery`), not Joi + `validatedQuery`. Parallel `countDocuments` + `find` matches other video-repo patterns (no `$facet` helper on this model).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

*No Critical/High findings.*

**Merge readiness:** No open backend Critical/High blockers on `sksh-307`. Post-merge with `main` (`e7d4f5d`) — ready to merge.
