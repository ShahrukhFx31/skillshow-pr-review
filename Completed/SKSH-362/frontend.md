# PR review — SKSH-362 (skillshow-admin-ui)

| Field | Value |
|-------|-------|
| PR | [#327](https://github.com/SkillshowFx/skillshow-admin-ui/pull/327) |
| Branch | `SKSH-362-1` → `main` |
| Head | `SKSH-362-1` @ `5fc79e9f` |
| Scope | Surface `publishResult.publishedBy` on distribution/video types; prefer stored name in video detail platform rows |
| Prompt | `pr-review/prompts/frontend-system-prompt.md` |
| Paired | Backend [#227](https://github.com/SkillshowFx/skillshow/pull/227), Orchestrator [#28](https://github.com/SkillshowFx/skillshow-distribution-orchestrator/pull/28) |

## GitHub comments

_No open inline findings._

## Findings

_No Critical, High, or Medium findings._

### Files reviewed

| File | Change |
|------|--------|
| `src/api/types/distribution.types.ts` | Add optional `publishedBy` on `publishResult` |
| `src/api/types/video.types.ts` | Add optional `publishedBy` on vendor log `publishResult` |
| `src/pages/videos/details/mappers.ts` | Prefer `log.publishResult?.publishedBy` before live connection metadata for `account` display |

### DRY / KISS / Global consistency scan

| Check | Verdict |
|-------|---------|
| Type contract aligned with backend/orchestrator `publishedBy` field | ✅ |
| Account display uses stored publish-time name (survives unlink) | ✅ `mapVendorLogToPlatform` |
| `buildNotPublishedPlatform` still reads live connection (correct — not yet published) | ✅ |
| List view (`mapBackendVideosToRows`) does not show account column | ✅ N/A |
| Protected modules | ✅ Not touched |

### Positive notes

- **KISS:** Single precedence change in `mapVendorLogToPlatform` — `publishedBy` first, then existing connection fallbacks.
- **Contract:** Types updated on both distribution poll and video detail payloads so `publishedBy` is typed end-to-end.
- **Backward compatible:** Older vendor logs without `publishedBy` still fall back to connection metadata.

### Optional follow-up (not reported as findings)

- `vendor-log-cache.utils.ts` `vendorLogEqualsForListPatch` omits `publishedBy`; harmless because completion polls set `externalUrl`/`externalId` in the same payload as `publishedBy`.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No Critical/High/Medium findings | — | — | — | — |

**Merge readiness:** No open Critical/High/Medium blockers.
