# PR review — SKSH-362 (skillshow-distribution-orchestrator)

| Field | Value |
|-------|-------|
| PR | [#28](https://github.com/SkillshowFx/skillshow-distribution-orchestrator/pull/28) |
| Branch | `SKSH-362` → `main` |
| Head | `SKSH-362` @ `280e627f` |
| Scope | Capture social account display name at publish time; persist on `VendorLog.publishResult.publishedBy` |
| Prompt | `pr-review/prompts/orchestrator-system-prompt.md` |
| Paired | Backend [#227](https://github.com/SkillshowFx/skillshow/pull/227), Frontend [#327](https://github.com/SkillshowFx/skillshow-admin-ui/pull/327) |

## GitHub comments

_No open inline findings._

## Findings

_No Critical, High, or Medium findings._

### Files reviewed

| File | Change |
|------|--------|
| `src/helpers/vendorConnection.helper.ts` | New `getVendorConnectionPublishedBy()` with shared `pickString` / `getVendorEntry` |
| `src/workers/PlatformDistributionWorker.ts` | Resolve `publishedBy` after successful publish; pass to `markCompleted` |
| `src/repositories/VendorLogRepository.ts` | Persist optional `publishedBy` on `publishResult` |
| `src/models/vendorLog.model.ts` | Schema field |
| `src/types/distribution.types.ts`, `src/types/vendor.types.ts` | Type + `VendorPublishResult` |
| `src/swagger/distribution.swagger.ts` | OpenAPI for `publishedBy` |
| `tests/helpers/vendorConnection.helper.test.ts` | Helper coverage |
| `tests/repositories/VendorLogRepository.test.ts` | `markCompleted` stores `publishedBy` |

### DRY / KISS / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuses existing connection lookup pattern (`getVendorEntry`, `pickString`) | ✅ |
| Single worker write path via `markCompleted` | ✅ |
| Swagger + types + repo + model updated together | ✅ Global consistency |
| Job failure / abort paths unchanged | ✅ No stuck-job risk |

### Positive notes

- **DRY:** `getVendorConnectionPublishedBy` mirrors `getVendorConnectionCredentials` structure; field key list covers common vendor metadata shapes (`profileUsername`, `pageName`, `displayName`, etc.).
- **Operational:** Value captured at publish completion so video details remain accurate after social account unlink.
- **Tests:** Helper + repository coverage for happy path and missing connection.

### Optional follow-up (not reported as findings)

- `publishedBy` is resolved **after** `vendorService.publish()`; if a user unlinks during a long publish, the name may be missing (rare). Resolving alongside credential load at job start would harden that edge case.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No Critical/High/Medium findings | — | — | — | — |

**Merge readiness:** No open Critical/High/Medium blockers.
