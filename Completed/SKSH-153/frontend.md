# Frontend PR Review — skillshow-admin-ui (`SKSH-153`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-153`  
**Base:** `main...HEAD`  
**HEAD:** `84996d4a` — refactor: update imports for distribute-eligibility utilities  
**Reviewed:** 2026-06-08 — full `/pr-review` re-review (entire diff)  
**Scope reviewed:** Full PR diff — **7 files**, ~111 insertions / ~79 deletions  
**Findings:** 0 Critical, 0 High, 3 Medium — **0 Open** (2 Fixed, 1 Accepted)  
**Note:** Review-only — findings for the developer; no agent code changes.

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/videos/utils/distribute-eligibility.utils.ts` | **New** — eligibility helpers, sort, published/publishing sets |
| `src/pages/videos/components/ConnectedPlatformSelector.tsx` | Distribute visibility via shared util; empty-state copy; `vendorLogs` prop |
| `src/pages/videos/details/components/distribute/DistributeModal.tsx` | Use shared eligibility utils; clear selection on open; pass `vendorLogs` |
| `src/pages/videos/details/components/distribute/utils.ts` | Remove duplicated eligibility/sort helpers (moved to shared util) |
| `src/pages/videos/details/index.tsx` | Disable footer Distribute when no eligible platforms |
| `src/pages/videos/types/video.types.ts` | Optional `vendorLogs` on `ConnectedPlatformSelectorProps` |
| `src/pages/videos/details/components/DistributeModal.tsx` | Import shared utils; pass `vendorLogs` (legacy modal) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Eligibility single source of truth (`getEligibleDistributePlatforms` + `getDistributeVisiblePlatformConfigs`) | ✅ Fixed — see #1 |
| Footer `distributeDisabled` and modal chip list use same util chain | ✅ |
| Shared util promoted to `src/pages/videos/utils/` | ✅ Fixed — see #3 |
| Both distribute modals + selector + details page use shared imports | ✅ Global consistency |
| Protected modules (`usePagination`, `DestructiveActionConfirmModal`, etc.) | ✅ Not touched |
| Server-list / destructive-action contracts | N/A — video distribute flow only |
| `cn()` / `@/ui` conventions | ✅ No issues |
| JSX conditional rendering | ✅ |

### Positive notes

- **DRY win:** `distribute-eligibility.utils.ts` centralizes published/publishing detection, sort order, and eligible-platform resolution; selector, footer, and both distribute modals share one module.
- **Re-review fixes:** `2ce606b5` unified selector visibility with `getDistributeVisiblePlatformConfigs`; `84996d4a` promoted helpers out of `details/components/distribute/utils.ts`.
- **UX:** Footer “Distribute to More Platform” disables when there are zero eligible targets; empty states distinguish no connections vs all connected platforms already used.
- **Intentional flow:** Banner-only connect during distribute (#2 Accepted); modal opens with empty selection; metadata prefills on chip selection.

---

---
Distribute eligibility rules duplicated (DRY / Global consistency)

Risk Level: MEDIUM  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 176-178  
File Path: src/pages/videos/utils/distribute-eligibility.utils.ts  
Lines: 54-75

Description:
**DRY / Global consistency:** Initial commit duplicated `connected ∩ ¬published ∩ ¬publishing` in the selector and footer. Re-review: `getDistributeVisiblePlatformConfigs` delegates to `getEligibleDistributePlatforms`, and distribute-mode `visiblePlatforms` calls that helper.

Impact:
- (Resolved) Footer disable and modal chips could previously diverge on eligibility rule changes.

Recommendation:
No further action — keep both call sites on shared utils.

**PR comment:** N/A — resolved in `2ce606b5`.

---

---
In-modal OAuth connect removed for unconnected platforms (UX / behavioral change)

Risk Level: MEDIUM  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 176-178, 335-346

Description:
On `main`, distribute appearance listed **all** `PLATFORM_CONFIG` entries; unconnected platforms rendered as “Connect +” chips. This PR shows only **eligible connected** platforms via `getDistributeVisiblePlatformConfigs`; users connect via page-level `PlatformConnectBanner`.

Impact:
- Workflow change: connecting a new platform during distribute requires closing the modal, using page banners, then reopening.
- `allowConnect` has no effect for unconnected platforms in distribute mode.

Recommendation:
No code change required. **Accepted** — product confirmed banner-only connect is intentional for SKSH-153.

**PR comment:** **Accepted** — banner-only connect during distribute is intentional per ticket owner.

---

---
Shared eligibility utils under page-specific distribute folder (file structure)

Risk Level: MEDIUM  
File Path: src/pages/videos/utils/distribute-eligibility.utils.ts  
Lines: 1-75  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 19

Description:
**KISS / file structure:** Initial re-review flagged `ConnectedPlatformSelector` importing from `details/components/distribute/utils.ts`. Commit `84996d4a` promotes eligibility helpers to `src/pages/videos/utils/distribute-eligibility.utils.ts`; selector, `details/index.tsx`, and both distribute modals import from there. Duplicates removed from `distribute/utils.ts`.

Impact:
- (Resolved) Inverted parent→nested-page dependency.

Recommendation:
No further action.

**PR comment:** N/A — resolved in `84996d4a`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Distribute eligibility rules duplicated | MEDIUM | ✅ Fixed | src/pages/videos/utils/distribute-eligibility.utils.ts | 54-75 |
| 2 | In-modal OAuth connect removed for unconnected platforms | MEDIUM | Accepted | src/pages/videos/components/ConnectedPlatformSelector.tsx | 176-178, 335-346 |
| 3 | Shared eligibility utils under page-specific distribute folder | MEDIUM | ✅ Fixed | src/pages/videos/utils/distribute-eligibility.utils.ts | 1-75 |

## GitHub review comments (ready to paste)

No open findings — prior threads can be resolved:

- **#1 / #3:** Fixed in `2ce606b5` + `84996d4a` (shared `distribute-eligibility.utils.ts`).
- **#2:** Accepted — banner-only connect intentional.

**Merge readiness:** **Merge-ready** — no open Critical/High/Medium blockers. All findings Fixed or Accepted.
