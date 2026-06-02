# SKSH-315 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-237` (SKSH-315)  
**Base:** `main`  
**Initial review:** 2026-06-01 — `sksh-237` vs `main`  
**Re-reviewed:** 2026-06-02 — `origin/sksh-237` @ `fbb91a14` (`fix: feedbacks`)  
**Latest re-review:** 2026-06-02 — `origin/sksh-237` @ `661bf30e` (merge `main` + layout/loading commits)  
**Scope:** Video detail sharing-site retry UX — unify with `useVideoDistributionRetry`, per-platform pending spinners, detail cache optimistic updates + vendor polling, mobile row actions.

## Overview

The PR replaces detail-page `useTransition` + `useOptimistic` + `useRetrySingleVendorMutation` with the shared `useVideoDistributionRetry` hook (already used on My Videos). Retry spinners are scoped per platform via `usePendingKey` / `useSyncExternalStore`. Detail React Query cache is patched optimistically and during vendor status polling (`replaceVendorLogInDetailVideo`). Page-level `startRetryPolling` interval refetch is removed in favor of hook-driven polls.

**SKSH-315 diff vs `main` (video scope, 13 files):** retry stack unchanged since `fbb91a14`; latest tip adds merged `main` work (e.g. `SkyPageLoading`, `actions` slot in `SharingSiteDetails` already on `main`) — not new retry regressions.

### Latest re-review verdict (`661bf30e`)

| # | Finding | Status |
|---|---------|--------|
| 1 | Silent retry guard | ✅ **Fixed** (unchanged) |
| 2 | List cache not patched from detail | **Accepted** (unchanged — team decision) |
| 3 | Mobile border without `cn()` | ✅ **Fixed** (unchanged) |
| 4 | Duplicate noop helpers | ✅ **Fixed** (unchanged) |

**New blockers:** None — no new Critical/High/Medium in SKSH-315 retry scope on latest tip.

---

## Findings

---
Silent retry guard drops user-facing error for missing job

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteDetails.tsx  
Lines: 36-43

Description:
(Initial.) `handleRetry` returned early when `!platform.distributionJobId` or `!platform.vendor` without feedback.

Impact:
- User taps retry on a failed row with no job id and sees no spinner, toast, or explanation.

Recommendation:
Delegate validation to `retryPlatform` (remove the silent guards), or mirror the hook toasts in `handleRetry`.

**Re-review (`fbb91a14`, `661bf30e`):** ✅ **Fixed** — `handleRetry` calls `retryPlatform`; hook toasts on missing job/vendor.

---

---
Detail-page retry does not patch My Videos list cache

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteDetails.tsx  
Lines: 27-29  
File Path: src/pages/videos/hooks/useVideoDistributionRetry.ts  
Lines: 117-118, 149-156

Description:
`SharingSiteDetails` calls `useVideoDistributionRetry({ detailQueryKey })` only. List view passes `listQueryFullKey`. After retry from video detail, list cache may lag until refetch.

Impact:
- My Videos table can show stale platform status after navigating back from detail.

Recommendation:
Pass `listQueryFullKey` or invalidate list queries on retry — deferred.

**Re-review (`661bf30e`):** **Accepted** — still detail-only; explicitly accepted out of PR scope (no code change on latest tip).

---

---
Conditional mobile border uses string concat instead of `cn()`

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteMobileCards.tsx  
Lines: 161-165

Description:
(Initial.) Template literal for conditional border instead of `cn()`.

**Re-review (`661bf30e`):** ✅ **Fixed** — uses `cn()`.

---

---
Duplicate noop retry-store fallbacks in row components

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/PlatformRow.tsx  
Lines: 31-34  
File Path: src/pages/videos/details/components/SharingSiteMobileCards.tsx  
Lines: 23-26

Description:
(Initial.) Copy-pasted noop helpers for pending retry store.

**Re-review (`661bf30e`):** ✅ **Fixed** — `NOOP_VIDEO_LIST_PENDING` reused.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Silent retry guard drops user-facing error for missing job | MEDIUM | ✅ Fixed | src/pages/videos/details/components/SharingSiteDetails.tsx | 36-43 |
| 2 | Detail-page retry does not patch My Videos list cache | MEDIUM | Accepted | src/pages/videos/details/components/SharingSiteDetails.tsx | 27-29 |
| 3 | Conditional mobile border uses string concat instead of `cn()` | MEDIUM | ✅ Fixed | src/pages/videos/details/components/SharingSiteMobileCards.tsx | 161-165 |
| 4 | Duplicate noop retry-store fallbacks in row components | MEDIUM | ✅ Fixed | src/pages/videos/details/components/PlatformRow.tsx | 31-34 |

**Positive notes:** Retry refactor stable across `fbb91a14` → `661bf30e`. Per-platform spinners and shared hook remain solid.

**Latest tip notes (not findings):** `isPolling` still unused in `SharingSiteDetails`. Merged commits touch loading/edit-request/video-table files outside SKSH-315 retry diff — no new retry issues found there.

**Merge readiness:** No open Critical/High/Medium blockers (1 Medium accepted). Frontend review complete for SKSH-315.
