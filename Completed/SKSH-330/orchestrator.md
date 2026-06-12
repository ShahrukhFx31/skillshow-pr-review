# Orchestrator PR Review — skillshow-distribution-orchestrator (`SKSH-330`)

**Repo:** skillshow-distribution-orchestrator  
**Branch:** `sksh-330`  
**Base:** `main...HEAD` @ `5598909f`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-11 (`5598909f` — `fix: feedback`)  
**Scope:** Social platform availability check in `PlatformDistributionWorker` (Critical / High only)  
**Prompts:** `orchestrator-system-prompt.md` (DRY / KISS / Global consistency)

**Aligned with:** [backend.md](./backend.md), [frontend.md](./frontend.md)

**Findings:** 1 (0 Critical, 1 High) — **0 Open**, **1 Fixed**

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/social-platform.constants.ts` | Managed vendors, unavailable message |
| `src/helpers/socialPlatformAvailability.helper.ts` | `isSocialPlatformActive` |
| `src/models/social-platform-setting.model.ts` | Shared Mongo model |
| `src/workers/PlatformDistributionWorker.ts` | Inactive-platform guard before publish |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Check in sole publish worker (`PlatformDistributionWorker`) | ✅ |
| Reads shared `SocialPlatformSetting` collection | ✅ |
| Early return without rethrow (no BullMQ retry loop) | ✅ KISS |
| Inactive path uses `abortDistribution` + `persistError` | ✅ Fixed |
| Other workers unchanged (post-publish polling) | ✅ Accepted |

### Positive notes

- **Re-review:** Inactive-platform failures now route through `abortDistribution` with `setJobFailed: false` before `updateOverallJobStatus`.
- **Defense in depth:** Blocks publish for queued jobs when admin disables a vendor after enqueue.

---

## GitHub comments (Open findings)

No open findings — prior comment resolved on branch.

---

## Findings

---
Inactive platform failure bypasses abortDistribution / persistError

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-distribution-orchestrator/src/workers/PlatformDistributionWorker.ts  
Lines: 74-99

Description:
**DRY / Contract.** Initial diff called `vendorLogRepo.markFailed` directly, skipping error-log persistence.

**Re-verification (`5598909f`):** ✅ **Fixed:**

```80:99:skillshow-distribution-orchestrator/src/workers/PlatformDistributionWorker.ts
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

**PR comment (`PlatformDistributionWorker.ts` line 80):** **Resolved** — inactive-platform failures use `abortDistribution` + `updateOverallJobStatus`, return without rethrow.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Inactive platform failure bypasses abortDistribution / persistError | HIGH | ✅ Fixed | skillshow-distribution-orchestrator/src/workers/PlatformDistributionWorker.ts | 74-99 | 80 |

**Merge readiness:** No open Critical/High blockers. Safe to merge from an orchestrator perspective.
