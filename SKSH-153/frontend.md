# Frontend PR Review — skillshow-admin-ui (`SKSH-153`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-153`  
**Base:** `main...HEAD`  
**HEAD:** `26b5cc3d` (merge main) + `e87d08bf` — refactor platform filtering  
**Reviewed:** 2026-06-08 — full `/pr-review` pass (entire diff)  
**Scope reviewed:** Full PR diff — **4 files**, ~59 insertions / ~35 deletions  
**Findings:** 0 Critical, 0 High, 2 Medium  
**Note:** Review-only — findings for the developer; no agent code changes.

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/videos/components/ConnectedPlatformSelector.tsx` | Distribute-mode visibility: connected-only, hide published/publishing; empty-state copy |
| `src/pages/videos/details/components/distribute/DistributeModal.tsx` | Extract published helper; clear selection on open (no auto-select); drop inline eligibility memo |
| `src/pages/videos/details/components/distribute/utils.ts` | `publishedPlatformsFromVendorLogs`, `getEligibleDistributePlatforms` |
| `src/pages/videos/details/index.tsx` | Disable footer Distribute when no eligible platforms |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Published/publishing platform sets centralized in `utils.ts` | ✅ |
| Footer `distributeDisabled` aligned with modal eligibility intent | ✅ (same rules, duplicated implementation — see #1) |
| Protected modules (`usePagination`, `DestructiveActionConfirmModal`, etc.) | ✅ Not touched |
| Server-list / destructive-action contracts | N/A — video distribute flow only |
| `cn()` / `@/ui` conventions | ✅ No issues |
| JSX conditional rendering | ✅ |
| Speculative abstraction | ✅ `getEligibleDistributePlatforms` is used by page + could back selector |

### Positive notes

- **DRY win:** `publishedPlatformsFromVendorLogs` replaces inline `useMemo` in `DistributeModal` and uses `BackendJobStatus.COMPLETED` consistently.
- **UX:** Footer “Distribute to More Platform” disables when there are zero eligible targets (`connected − published − publishing`), avoiding a dead-end modal.
- **Empty states:** Distribute mode now distinguishes “no connections” vs “all connected platforms already distributed/publishing” with clearer copy.
- **Intentional flow change:** Modal opens with no pre-selected platforms; metadata prefills on user selection via existing `handleSelectedPlatformsChange` — simpler configure step, fewer surprise submissions.

---

---
Distribute eligibility rules duplicated (DRY / Global consistency)

Risk Level: MEDIUM  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 174-194  
File Path: src/pages/videos/details/components/distribute/utils.ts  
Lines: 115-125  
File Path: src/pages/videos/details/index.tsx  
Lines: 196-202

Description:
**DRY / Global consistency:** The same eligibility predicate — `connected ∩ ¬published ∩ ¬publishing` — is implemented twice: in `getEligibleDistributePlatforms` (footer disable + shared util) and again inside `ConnectedPlatformSelector`’s `visiblePlatforms` distribute branch. The filter chains are parallel today but not wired through one helper.

Impact:
- A future tweak (e.g. new in-flight status, partial publish rules) applied in only one place can leave the footer enabled while the modal shows an empty selector (or vice versa).
- Reviewers must diff two filter implementations on every eligibility change.

Recommendation:
Drive distribute visibility from the shared util — e.g. export `getEligibleDistributePlatforms` (or a `getDistributeVisiblePlatformConfigs(connections, vendorLogs)` that maps keys back to `PLATFORM_CONFIG`) and use it in `ConnectedPlatformSelector` when `appearance === "distribute"`. Keep `index.tsx` on the same function for `distributeDisabled`.

**PR comment (`ConnectedPlatformSelector.tsx` line 176):**  
**Medium (DRY / Global consistency):** Distribute `visiblePlatforms` repeats the same connected/published/publishing filters as `getEligibleDistributePlatforms` — please reuse the shared util so the footer disable state and chip list cannot drift.

---

---
In-modal OAuth connect removed for unconnected platforms (UX / behavioral change)

Risk Level: MEDIUM  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 176-183, 340-346

Description:
On `main`, distribute appearance listed **all** `PLATFORM_CONFIG` entries; unconnected platforms rendered as “Connect +” chips and called `connectVendor` from the modal. This PR filters distribute mode to **connected-only** chips, so `connectVendor` is unreachable for new platforms while the modal is open. Users are directed to page-level `PlatformConnectBanner` copy instead.

Impact:
- Workflow change: adding a platform mid-distribute requires closing the modal, connecting via banners, then reopening — previously possible in-place from chips.
- `allowConnect` is still passed into the selector but has no effect for unconnected platforms in distribute mode (dead path).

Recommendation:
If banner-only connect is the intended SKSH-153 design, document in ticket/QA and consider removing or narrowing `allowConnect` for distribute appearance to avoid misleading API surface. If in-modal connect should remain, show unconnected platforms in distribute mode (connected filter only for eligibility/disable, not visibility) or add an explicit “Connect another platform” affordance inside the modal.

**PR comment (`ConnectedPlatformSelector.tsx` line 177):**  
**Medium (UX):** Distribute mode now hides unconnected platforms entirely — users can’t OAuth-connect from chips inside the modal anymore. Please confirm this is intentional vs `main` (banner-only connect) or restore Connect+ chips for unconnected vendors.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Distribute eligibility rules duplicated | MEDIUM | Open | src/pages/videos/components/ConnectedPlatformSelector.tsx | 174-194 |
| 2 | In-modal OAuth connect removed for unconnected platforms | MEDIUM | Open | src/pages/videos/components/ConnectedPlatformSelector.tsx | 176-183, 340-346 |

## GitHub review comments (ready to paste)

**`ConnectedPlatformSelector.tsx` ~line 176**  
Medium (DRY / Global consistency): Distribute `visiblePlatforms` duplicates `getEligibleDistributePlatforms` filters. Reuse the shared util so footer disable and chip list stay aligned when eligibility rules change.

**`ConnectedPlatformSelector.tsx` ~line 177**  
Medium (UX): Distribute mode no longer shows unconnected “Connect +” chips — users must leave the modal to connect via page banners. Confirm intentional for SKSH-153 or restore in-modal connect for unconnected platforms.

**Merge readiness:** **Merge-ready with minor follow-ups** — no Critical/High blockers; 2 open Medium items (DRY consolidation + confirm in-modal connect UX). Safe to merge if product accepts banner-only connect during distribute.
