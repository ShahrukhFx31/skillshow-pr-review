# SKSH-315 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-237` (SKSH-315)  
**Base:** `main`  
**Reviewed:** 2026-06-01 — `sksh-237` vs `main`  
**Scope:** Video detail sharing-site retry UX — unify with `useVideoDistributionRetry`, per-platform pending spinners, detail cache optimistic updates + vendor polling, mobile row actions.

## Overview

The PR replaces detail-page `useTransition` + `useOptimistic` + `useRetrySingleVendorMutation` with the shared `useVideoDistributionRetry` hook (already used on My Videos). Retry spinners are scoped per platform via `usePendingKey` / `useSyncExternalStore`. Detail React Query cache is patched optimistically and during vendor status polling (`replaceVendorLogInDetailVideo`). Page-level `startRetryPolling` interval refetch is removed in favor of hook-driven polls.

**Files touched (9):** `PlatformRow`, `SharingSiteDetails`, `SharingSiteMobileCards`, `VideoInfoCard`, `details/index`, `interfaces`, `useVideoDistributionRetry`, `video-list.types`, `vendor-log-cache.utils`.

---

## Findings

---
Silent retry guard drops user-facing error for missing job

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteDetails.tsx  
Lines: 35-42

Description:
`handleRetry` returns early when `!platform.distributionJobId` or `!platform.vendor` without feedback. The previous implementation called `message.error("Cannot retry: distribution job not found.")` before mutating. `retryPlatform` in the hook does toast for missing `distributionJobId`, but that path is never reached because `handleRetry` bails first.

Impact:
- User taps retry on a failed row with no job id and sees no spinner, toast, or explanation.
- Harder to diagnose data/API issues during QA or support.

Recommendation:
Delegate validation to `retryPlatform` (remove the silent guards), or mirror the hook toasts in `handleRetry`:

```ts
if (!platform.distributionJobId) {
  toast.error("Cannot retry: distribution job not found.");
  return;
}
```

**PR comment (`SharingSiteDetails.tsx` line 36):**  
**Medium:** `handleRetry` now returns silently when `distributionJobId` / `vendor` is missing; please restore the previous `message.error` (or call `retryPlatform` so the hook toast runs) so retry failures are visible.

---

---
Detail-page retry does not patch My Videos list cache

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteDetails.tsx  
Lines: 26-28  
File Path: src/pages/videos/details/index.tsx  
Lines: 203-219  
File Path: src/pages/videos/hooks/useVideoDistributionRetry.ts  
Lines: 771-798

Description:
`SharingSiteDetails` calls `useVideoDistributionRetry({ detailQueryKey })` only. List view passes `listQueryFullKey`. After retry from video detail, optimistic updates and poll patches apply to the detail query; the shared list cache for the same video is unchanged until a full refetch or navigation-driven invalidation.

Impact:
- Athlete returns to My Videos and may still see **Failed** on a platform row while detail already shows **Retrying** / **Published**.
- Inconsistent cross-route state until background refetch or manual refresh.

Recommendation:
If list and detail can be open in the same session, pass both keys from the parent when the list query is known (or invalidate `VIDEO_LIST` query keys on retry success). Minimal option: `void queryClient.invalidateQueries({ queryKey: [...queryKeyRoot, VIDEO_LIST_QUERY_KEY_SEGMENT.ALL] })` in `onSuccess` when only detail key is configured.

**PR comment (`SharingSiteDetails.tsx` line 26):**  
**Medium:** Detail retry only patches `detailQueryKey`; consider also updating or invalidating the My Videos list cache so status stays consistent when users navigate back to the table.

---

---
Conditional mobile border uses string concat instead of `cn()`

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/SharingSiteMobileCards.tsx  
Lines: 168-171

Description:
Expanded-row border styling uses a template literal with `isExpanded && "border-b …"` instead of `cn()` from `@/utils`, which is the project convention for conditional Tailwind classes.

Impact:
- Inconsistent with surrounding components (`PlatformRow` uses `cn`).
- Harder to extend with additional conditional utilities without class conflicts.

Recommendation:

```tsx
import { cn } from "@/utils";

<div
  className={cn(
    "mt-3 flex min-w-0 flex-row items-center justify-between gap-2 pb-3",
    isExpanded && "border-b border-border/70",
  )}
>
```

**PR comment (`SharingSiteMobileCards.tsx` line 168):**  
Please use `cn()` for the conditional `border-b` classes to match repo Tailwind conventions.

---

---
Duplicate noop retry-store fallbacks in row components

Risk Level: MEDIUM  
File Path: src/pages/videos/details/components/PlatformRow.tsx  
Lines: 13-19, 38-42  
File Path: src/pages/videos/details/components/SharingSiteMobileCards.tsx  
Lines: 14-20, 30-34

Description:
Identical `noopIsRetrying` / `noopSubscribeRetrying` helpers are copy-pasted in `PlatformRow` and `SharingSiteMobileCards` (`MobileRetryButton`), then passed into `usePendingKey` as fallbacks when optional `isRetrying` / `subscribeRetrying` props are omitted.

Impact:
- Two places to update if the pending-store contract changes.
- Noise in presentational components.

Recommendation:
Extract to `src/pages/videos/utils/retry-pending-noops.ts` (or colocate next to `usePendingKey`), or reuse `NOOP_VIDEO_LIST_PENDING.isRetrying` / `.subscribeRetrying` from `videoListPendingContext.ts` instead of duplicating helpers.

**PR comment (`PlatformRow.tsx` line 40):**  
**Medium:** `noopIsRetrying` / `noopSubscribeRetrying` are duplicated in `SharingSiteMobileCards.tsx` (`MobileRetryButton`, line 32). Please extract shared noops (e.g. `retry-pending-noops.ts` next to `usePendingKey`) or reuse `NOOP_VIDEO_LIST_PENDING` from `videoListPendingContext.ts`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Silent retry guard drops user-facing error for missing job | MEDIUM | Open | src/pages/videos/details/components/SharingSiteDetails.tsx | 35-42 |
| 2 | Detail-page retry does not patch My Videos list cache | MEDIUM | Open | src/pages/videos/details/components/SharingSiteDetails.tsx | 26-28 |
| 3 | Conditional mobile border uses string concat instead of `cn()` | MEDIUM | Open | src/pages/videos/details/components/SharingSiteMobileCards.tsx | 168-171 |
| 4 | Duplicate noop retry-store fallbacks in row components | MEDIUM | Open | src/pages/videos/details/components/PlatformRow.tsx | 38-42 |

**Positive notes:** Per-platform spinners fix the prior global `isPending` bug (all retry icons spun together). Reusing `useVideoDistributionRetry` + `usePendingKey` aligns detail with list behavior. Detail cache optimistic + poll patching removes brittle page-level `setInterval` refetch. Mobile hides expand during `PROCESSING` / `RETRYING`, matching desktop semantics.

**Merge readiness:** No Critical/High blockers; 4 open Medium items (UX consistency and conventions).
