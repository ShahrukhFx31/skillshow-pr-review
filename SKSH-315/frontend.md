# SKSH-315 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-237` (SKSH-315)  
**Base:** `main`  
**Initial review:** 2026-06-01 — `sksh-237` vs `main`  
**Re-reviewed:** 2026-06-02 — `origin/sksh-237` @ `fbb91a14` (`fix: feedbacks`)  
**Scope:** Video detail sharing-site retry UX — unify with `useVideoDistributionRetry`, per-platform pending spinners, detail cache optimistic updates + vendor polling, mobile row actions.

## Overview

The PR replaces detail-page `useTransition` + `useOptimistic` + `useRetrySingleVendorMutation` with the shared `useVideoDistributionRetry` hook (already used on My Videos). Retry spinners are scoped per platform via `usePendingKey` / `useSyncExternalStore`. Detail React Query cache is patched optimistically and during vendor status polling (`replaceVendorLogInDetailVideo`). Page-level `startRetryPolling` interval refetch is removed in favor of hook-driven polls.

**Files touched (14 vs `main`, incl. newline-only):** core video detail retry stack + `NOOP_VIDEO_LIST_PENDING` reuse in row components.

### Re-review verdict (`fbb91a14`)

| # | Original finding | Status |
|---|------------------|--------|
| 1 | Silent retry guard | ✅ **Fixed** — `handleRetry` delegates to `retryPlatform`; hook toasts on missing job/vendor |
| 2 | List cache not patched from detail | **Accepted** — acknowledged as acceptable for current scope |
| 3 | Mobile border without `cn()` | ✅ **Fixed** |
| 4 | Duplicate noop helpers | ✅ **Fixed** — `NOOP_VIDEO_LIST_PENDING` from `videoListPendingContext.ts` |

**New blockers:** None (no new Critical/High/Medium introduced in `fix: feedbacks`).

---

## Findings

---
Silent retry guard drops user-facing error for missing job

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteDetails.tsx  
Lines: 35-42

Description:
(Initial.) `handleRetry` returned early when `!platform.distributionJobId` or `!platform.vendor` without feedback.

Impact:
- User taps retry on a failed row with no job id and sees no spinner, toast, or explanation.

Recommendation:
Delegate validation to `retryPlatform` (remove the silent guards), or mirror the hook toasts in `handleRetry`.

**Re-review:** ✅ **Fixed** — guards removed; `handleRetry` always calls `retryPlatform` with `platform.distributionJobId ?? ""` / `platform.vendor ?? ""`. `retryPlatform` (lines 267–273 in `useVideoDistributionRetry.ts`) shows toasts for empty vendor or missing `distributionJobId`.

---

---
Detail-page retry does not patch My Videos list cache

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteDetails.tsx  
Lines: 26-28  
File Path: src/pages/videos/details/index.tsx  
Lines: 203-219  
File Path: src/pages/videos/hooks/useVideoDistributionRetry.ts  
Lines: 117-118, 149-156

Description:
`SharingSiteDetails` calls `useVideoDistributionRetry({ detailQueryKey })` only. List view passes `listQueryFullKey`. After retry from video detail, optimistic updates and poll patches apply to the detail query; the shared list cache for the same video is unchanged until a full refetch or navigation-driven invalidation.

Impact:
- Athlete returns to My Videos and may still see **Failed** on a platform row while detail already shows **Retrying** / **Published**.
- Inconsistent cross-route state until background refetch or manual refresh.

Recommendation:
Pass both `detailQueryKey` and `listQueryFullKey` when the list query is available, or `invalidateQueries` on the video list root (e.g. `["videos"]`) on retry success / terminal poll when only detail key is configured.

**Re-review:** **Accepted** — still present on `fbb91a14` (no `listQueryFullKey` / list invalidation), but explicitly accepted as out of current PR scope.

**PR comment (`SharingSiteDetails.tsx` line 26):**  
**Medium (accepted):** Detail retry still only patches `detailQueryKey`. Cache-sync follow-up is accepted for now and can be handled in a separate improvement PR.

---

---
Conditional mobile border uses string concat instead of `cn()`

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteMobileCards.tsx  
Lines: 161-165

Description:
(Initial.) Expanded-row border styling used a template literal instead of `cn()`.

Impact:
- Inconsistent with project Tailwind conventions.

Recommendation:
Use `cn()` for conditional `border-b` classes.

**Re-review:** ✅ **Fixed** — `cn("mt-3 flex …", isExpanded && "border-b border-border/70")`.

---

---
Duplicate noop retry-store fallbacks in row components

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/PlatformRow.tsx  
Lines: 31-34  
File Path: src/pages/videos/details/components/SharingSiteMobileCards.tsx  
Lines: 23-26

Description:
(Initial.) Identical `noopIsRetrying` / `noopSubscribeRetrying` helpers were copy-pasted in `PlatformRow` and `SharingSiteMobileCards`.

Impact:
- Two places to update if the pending-store contract changes.

Recommendation:
Reuse `NOOP_VIDEO_LIST_PENDING` from `videoListPendingContext.ts` or extract shared noops.

**Re-review:** ✅ **Fixed** — both components use `NOOP_VIDEO_LIST_PENDING.isRetrying` / `.subscribeRetrying`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Silent retry guard drops user-facing error for missing job | MEDIUM | ✅ Fixed | src/pages/videos/details/components/SharingSiteDetails.tsx | 35-42 |
| 2 | Detail-page retry does not patch My Videos list cache | MEDIUM | Accepted | src/pages/videos/details/components/SharingSiteDetails.tsx | 26-28 |
| 3 | Conditional mobile border uses string concat instead of `cn()` | MEDIUM | ✅ Fixed | src/pages/videos/details/components/SharingSiteMobileCards.tsx | 161-165 |
| 4 | Duplicate noop retry-store fallbacks in row components | MEDIUM | ✅ Fixed | src/pages/videos/details/components/PlatformRow.tsx | 31-34 |

**Positive notes:** Per-platform spinners fix the prior global `isPending` bug. Shared hook + detail cache polling is a solid architecture improvement. `fix: feedbacks` addressed three of four review items cleanly via delegation to `retryPlatform`, `cn()`, and `NOOP_VIDEO_LIST_PENDING`.

**Re-review notes (not raised as findings):** `isPolling` is still passed from `details/index.tsx` but unused in `SharingSiteDetails` (pre-existing dead prop). PR also includes newline-at-EOF-only touches on `TeamTablePaginationBar`, `MobilePlatformCard`, `antd.adapter` — no functional change.

**Merge readiness:** No open Critical/High/Medium blockers (1 Medium accepted).
