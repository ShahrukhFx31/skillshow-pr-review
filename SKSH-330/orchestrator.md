# Orchestrator PR Review — skillshow-distribution-orchestrator (`SKSH-330`)

**Repo:** skillshow-distribution-orchestrator  
**Branch:** `sksh-330`  
**Base:** `main...HEAD` @ `55ce7dcd`  
**Initial review:** 2026-06-11  
**Scope:** Read social-platform availability from shared Mongo collection; block publish in `PlatformDistributionWorker` when admin-disabled (Critical / High only)  
**Prompts:** `orchestrator-system-prompt.md` (DRY / KISS / Global consistency)

**Aligned with:** [backend.md](./backend.md), [frontend.md](./frontend.md)

**Findings:** 1 (0 Critical, 1 High) — **1 Open**

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/social-platform.constants.ts` | `MANAGED_SOCIAL_VENDORS`, unavailable message |
| `src/helpers/socialPlatformAvailability.helper.ts` | `isSocialPlatformActive`, x → twitter normalization |
| `src/models/social-platform-setting.model.ts` | Mongoose model (reads same collection as main API) |
| `src/workers/PlatformDistributionWorker.ts` | Early-exit when platform inactive |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Availability check placed in `PlatformDistributionWorker` (sole publish worker) | ✅ Correct layer |
| Reads shared `SocialPlatformSetting` Mongo collection seeded by main API | ✅ |
| `MANAGED_SOCIAL_VENDORS` + x alias aligned with skillshow backend/frontend | ✅ |
| Early return without rethrow (avoids BullMQ retry loop on business rule) | ✅ KISS |
| Inactive path uses `markFailed` only, skips `abortDistribution` / `persistError` | ❌ Contract — see #1 |
| `VendorStatusCheckWorker` / `VendorDataFetchWorker` unchanged | ✅ Accepted — post-publish polling, not new publishes |
| `createWorkerEventHandlers` / `updateOverallJobStatus` patterns unchanged | ✅ |

### Positive notes

- **Defense in depth:** Even if a platform job is already enqueued when admin disables a vendor, the worker blocks publish before S3 download / vendor HTTP.
- **Job lifecycle:** Returning without throw after `markFailed` + `updateOverallJobStatus` correctly avoids pointless BullMQ retries for a permanent business failure.
- **Defaults:** Missing DB row defaults to active (`row?.isActive ?? true`), matching backend semantics.

---

## GitHub comments (Open findings)

### 1. `skillshow-distribution-orchestrator/src/workers/PlatformDistributionWorker.ts` line 80

**PR comment (line 80):** **High (Contract):** When a platform is admin-disabled, this path calls `vendorLogRepo.markFailed` directly and skips `abortDistribution` / `persistError`. Other failure paths in this worker use `abortDistribution` with `setJobFailed: false` so errors land in the error-log collection. Please route inactive-platform failures through `abortDistribution` (synthetic `Error` + `ERROR_SOURCE.PLATFORM_DISTRIBUTION` / `ERROR_STAGE.VENDOR_PUBLISH`) before `updateOverallJobStatus`, still returning without rethrow so BullMQ does not retry a permanent business failure.

---

## Findings

---
Inactive platform failure bypasses abortDistribution / persistError

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-distribution-orchestrator/src/workers/PlatformDistributionWorker.ts  
Lines: 74-89

Description:
**DRY / Contract.** When `isSocialPlatformActive(vendor)` is false, the worker calls `vendorLogRepo.markFailed` and `updateOverallJobStatus` directly. Every other failure path in `PlatformDistributionWorker` uses `abortDistribution`, which writes to the error-log collection via `persistError` before marking the vendor log failed (see `abortDistribution.helper.ts`).

```74:89:skillshow-distribution-orchestrator/src/workers/PlatformDistributionWorker.ts
      const platformActive = await isSocialPlatformActive(vendor);
      if (!platformActive) {
        logger.warn(
          "PlatformDistributionWorker.process: platform inactive, skipping publish",
          { vendor, distributionJobId, vendorLogId }
        );
        await this.vendorLogRepo.markFailed(
          vendorLogId,
          SOCIAL_PLATFORM_UNAVAILABLE_MESSAGE
        );
        await updateOverallJobStatus(
          distributionJobId,
          this.distributionJobRepo,
          this.vendorLogRepo
        );
        return;
      }
```

Impact:
- Inactive-platform failures are invisible in the orchestrator error-log store — ops/debugging must rely on vendor log `lastError` only.
- Divergent failure persistence pattern in the same worker will drift (e.g. missing `ERROR_SOURCE`, `ERROR_STAGE`, `bullJobId` context).

Recommendation:
Replace the `markFailed` call at line 80 with `abortDistribution` (`setJobFailed: false`), then keep `updateOverallJobStatus` and `return`:

```typescript
await abortDistribution(
  {
    source: ERROR_SOURCE.PLATFORM_DISTRIBUTION,
    stage: ERROR_STAGE.VENDOR_PUBLISH,
    error: new Error(SOCIAL_PLATFORM_UNAVAILABLE_MESSAGE),
    distributionJobId,
    vendorLogId,
    vendor,
    ...(job.id != null ? { bullJobId: job.id.toString() } : {}),
  },
  this.distributionJobRepo,
  this.vendorLogRepo,
  { setJobFailed: false }
);
await updateOverallJobStatus(
  distributionJobId,
  this.distributionJobRepo,
  this.vendorLogRepo
);
return;
```

**PR comment (`PlatformDistributionWorker.ts` line 80):** **High (Contract):** When a platform is admin-disabled, this path calls `vendorLogRepo.markFailed` directly and skips `abortDistribution` / `persistError`. Other failure paths in this worker use `abortDistribution` with `setJobFailed: false` so errors land in the error-log collection. Please route inactive-platform failures through `abortDistribution` (synthetic `Error` + `ERROR_SOURCE.PLATFORM_DISTRIBUTION` / `ERROR_STAGE.VENDOR_PUBLISH`) before `updateOverallJobStatus`, still returning without rethrow so BullMQ does not retry a permanent business failure.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Inactive platform failure bypasses abortDistribution / persistError | HIGH | Open | skillshow-distribution-orchestrator/src/workers/PlatformDistributionWorker.ts | 74-89 | 80 |

**Merge readiness:** **Not merge-ready** — 1 open High finding (error persistence contract). Fix before merge.
