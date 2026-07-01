# PR review (SKSH-372) — skillshow

PR: `https://github.com/SkillshowFx/skillshow/pull/224`  
Base: `main`  
Head: `SKSH-372` @ `70ce8af`

Prompt: `pr-review/prompts/backend-system-prompt.md`

**Aligned with:** [frontend.md](./frontend.md)

## GitHub comments

No inline comments — no Open Critical/High findings.

## Findings

No Critical or High findings.

### Positive notes

- **Root cause fix:** `computeVideoStats` now uses `buildMyVideosListScopeQuery` (same scope as `GET /v1/videos`) instead of `getMyVideosMutationScope` / `buildMyVideosAccessScopeQuery`, which still included owned `isEditUploaded` rows. This aligns dashboard `total` with My Videos.
- **Global consistency:** `VIDEO_EXCLUDE_EDIT_UPLOADED_QUERY` applied in `countByUser`, `findRecentByUserLean`, and `linkedAthletesVideoFilter` — consistent with SKSH-245 scope utilities.
- **Parent dashboard KPI:** Parent `myUploadedVideos` count switched from raw `countByUser` to `buildMyVideosListScopeQuery` + `countByFilter`, excluding edit-request source uploads.
- **Cache hygiene:** New `invalidateVideoStatsCacheForUser` uses per-user `buildCacheKey` (not namespace flush) and is called after `recordVideo`; matches security audit guidance for scoped cache invalidation.
- **Tests:** `parent-link.service.test.ts` mocks updated for the new count path.

### Optional follow-up (not reported as findings)

- Consider invalidating video-stats cache on regular `POST /v1/videos` create and `deleteUploadedVideo` as well, so `DistributionOverview` refreshes within the 120s TTL window (pre-existing gap, not introduced here).
- `recordVideo` invalidation is low value after the scope fix (edit uploads no longer affect stats) but harmless during rollout if stale inflated cache keys exist.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No Critical/High findings | — | — | — | — |

**Merge readiness:** No open Critical/High blockers.
