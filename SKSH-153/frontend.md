# Frontend PR Review — skillshow-admin-ui (`SKSH-153`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-153`  
**Base:** `main...HEAD`  
**HEAD:** `2ce606b5` — feat: enhance ConnectedPlatformSelector and DistributeModal with vendorLogs support  
**Reviewed:** 2026-06-08 — full `/pr-review` re-review (entire diff)  
**Scope reviewed:** Full PR diff — **6 files**, ~68 insertions / ~34 deletions  
**Findings:** 0 Critical, 0 High, 2 Medium (1 Fixed, 1 Open, 1 Open)  
**Note:** Review-only — findings for the developer; no agent code changes.

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/videos/components/ConnectedPlatformSelector.tsx` | Distribute visibility via `getDistributeVisiblePlatformConfigs`; empty-state copy; `vendorLogs` prop |
| `src/pages/videos/details/components/distribute/DistributeModal.tsx` | Extract published helper; clear selection on open; pass `vendorLogs` to selector |
| `src/pages/videos/details/components/distribute/utils.ts` | `publishedPlatformsFromVendorLogs`, `getEligibleDistributePlatforms`, `getDistributeVisiblePlatformConfigs` |
| `src/pages/videos/details/index.tsx` | Disable footer Distribute when no eligible platforms |
| `src/pages/videos/types/video.types.ts` | Optional `vendorLogs` on `ConnectedPlatformSelectorProps` |
| `src/pages/videos/details/components/DistributeModal.tsx` | Pass `vendorLogs` (legacy modal — not used by `index.tsx`) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Eligibility single source of truth (`getEligibleDistributePlatforms` + `getDistributeVisiblePlatformConfigs`) | ✅ Fixed — see #1 |
| Footer `distributeDisabled` and modal chip list use same util chain | ✅ |
| Protected modules (`usePagination`, `DestructiveActionConfirmModal`, etc.) | ✅ Not touched |
| Server-list / destructive-action contracts | N/A — video distribute flow only |
| Shared util placement vs import direction | ⚠️ See #3 |
| `cn()` / `@/ui` conventions | ✅ No issues |
| JSX conditional rendering | ✅ |

### Positive notes

- **DRY win:** `publishedPlatformsFromVendorLogs`, `getEligibleDistributePlatforms`, and `getDistributeVisiblePlatformConfigs` centralize distribute eligibility; selector and footer cannot drift on filter rules.
- **Re-review fix:** Commit `2ce606b5` wires `ConnectedPlatformSelector` distribute mode through `getDistributeVisiblePlatformConfigs(connections, vendorLogs)` — resolves prior duplication finding.
- **UX:** Footer “Distribute to More Platform” disables when there are zero eligible targets; empty states distinguish no connections vs all connected platforms already used.
- **Intentional flow:** Modal opens with no pre-selected platforms; metadata prefills when the user selects chips via `handleSelectedPlatformsChange`.

---

---
Distribute eligibility rules duplicated (DRY / Global consistency)

Risk Level: MEDIUM  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 176-178  
File Path: src/pages/videos/details/components/distribute/utils.ts  
Lines: 115-135

Description:
**DRY / Global consistency:** Initial commit duplicated `connected ∩ ¬published ∩ ¬publishing` in the selector and in `getEligibleDistributePlatforms`. Re-review: `getDistributeVisiblePlatformConfigs` now delegates to `getEligibleDistributePlatforms`, and distribute-mode `visiblePlatforms` calls that helper.

Impact:
- (Resolved) Footer disable and modal chips could previously diverge on eligibility rule changes.

Recommendation:
No further action — keep both call sites on `getEligibleDistributePlatforms` / `getDistributeVisiblePlatformConfigs`.

**PR comment:** N/A — resolved in `2ce606b5`.

---

---
In-modal OAuth connect removed for unconnected platforms (UX / behavioral change)

Risk Level: MEDIUM  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 176-178, 335-346

Description:
On `main`, distribute appearance listed **all** `PLATFORM_CONFIG` entries; unconnected platforms rendered as “Connect +” chips and called `connectVendor` from the modal. This PR shows only **eligible connected** platforms (via `getDistributeVisiblePlatformConfigs`), so in-modal OAuth for new vendors is unavailable. Users are directed to page-level `PlatformConnectBanner` copy in the empty state.

Impact:
- Workflow change: connecting a new platform during distribute requires closing the modal, using page banners, then reopening.
- `allowConnect` is still passed into the selector but has no effect for unconnected platforms in distribute mode.

Recommendation:
If banner-only connect is the intended SKSH-153 design, mark **Accepted** in QA/ticket notes and consider documenting in component JSDoc. If in-modal connect should remain, extend distribute visibility to include unconnected `PLATFORM_CONFIG` rows (Connect+ chips) while keeping eligibility/disable logic on the shared util for the footer.

**PR comment (`ConnectedPlatformSelector.tsx` line 177):**  
**Medium (UX):** Distribute mode only shows eligible connected platforms — no in-modal “Connect +” for unconnected vendors. Please confirm banner-only connect is intentional for SKSH-153 or restore Connect+ chips for unconnected platforms.

---

---
Shared eligibility utils live under page-specific distribute folder (file structure)

Risk Level: MEDIUM  
File Path: src/pages/videos/components/ConnectedPlatformSelector.tsx  
Lines: 19  
File Path: src/pages/videos/details/components/distribute/utils.ts  
Lines: 115-135  
File Path: src/pages/videos/details/index.tsx  
Lines: 33

Description:
**KISS / file structure:** `ConnectedPlatformSelector` (`src/pages/videos/components/`) now imports `getDistributeVisiblePlatformConfigs` from `src/pages/videos/details/components/distribute/utils.ts`. Eligibility helpers are consumed by a shared videos component, the video details page, and both distribute modals — they are no longer details-only.

Impact:
- Inverted dependency: a parent/shared component imports from a nested page subfolder, making reuse and testing harder if upload or other routes need the same rules later.
- Future moves of `details/` risk breaking upload/distribute shared UI silently.

Recommendation:
Promote `publishedPlatformsFromVendorLogs`, `getEligibleDistributePlatforms`, and `getDistributeVisiblePlatformConfigs` to `src/pages/videos/utils/distribute-eligibility.utils.ts` (or similar) and import from there in selector, `index.tsx`, and distribute modals. Keep distribute-modal-specific helpers in `details/components/distribute/utils.ts`.

**PR comment (`ConnectedPlatformSelector.tsx` line 19):**  
**Medium (file structure):** Shared selector imports distribute eligibility from `details/components/distribute/utils` — consider promoting these helpers to `src/pages/videos/utils/` since they’re used outside the details page subtree.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Distribute eligibility rules duplicated | MEDIUM | ✅ Fixed | src/pages/videos/details/components/distribute/utils.ts | 115-135 |
| 2 | In-modal OAuth connect removed for unconnected platforms | MEDIUM | Open | src/pages/videos/components/ConnectedPlatformSelector.tsx | 176-178, 335-346 |
| 3 | Shared eligibility utils under page-specific distribute folder | MEDIUM | Open | src/pages/videos/components/ConnectedPlatformSelector.tsx | 19 |

## GitHub review comments (ready to paste)

**`ConnectedPlatformSelector.tsx` ~line 177**  
Medium (UX): Distribute mode only shows eligible connected platforms — no in-modal “Connect +” for unconnected vendors. Confirm banner-only connect is intentional for SKSH-153 or restore Connect+ chips for unconnected platforms.

**`ConnectedPlatformSelector.tsx` ~line 19**  
Medium (file structure): Shared selector imports eligibility from `details/components/distribute/utils`. Promote `getEligibleDistributePlatforms` / `getDistributeVisiblePlatformConfigs` to `src/pages/videos/utils/` since they’re used outside the details subtree.

**Merge readiness:** **Merge-ready with minor follow-ups** — no Critical/High blockers; eligibility DRY is fixed (`2ce606b5`). Two open Medium items: confirm in-modal connect UX (#2) and optional util path promotion (#3). Safe to merge if product accepts banner-only connect during distribute.
