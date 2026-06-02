# Frontend PR Re-Review — skillshow-admin-ui (`SKSH-268`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-268`  
**Base:** `main...HEAD`  
**Scope:** Re-check previously reported frontend findings + scan for newly introduced frontend issues. Pagination excluded per request.

**Result:** 2 remaining findings → **1 Partially fixed, 1 Accepted**.  
**New issues introduced:** **None (Critical/High/Medium)** in the latest fixes.

---

---
Desktop-only `modalClassNames` / `modalStyles` on responsive modals

Risk Level: MEDIUM
File Path: src/pages/auth/components/Agerestrictionmodal.tsx
Lines: 35-38
File Path: src/pages/videos/details/components/distribute/DistributeModal.tsx
Lines: 349-367

Description:
Accepted for now. `DistributeModal` now applies both `drawerClassNames`/`drawerStyles` and `modalClassNames`/`modalStyles`, and for `Agerestrictionmodal` the current behavior is intentional because the team is standardizing on Tailwind breakpoints rather than custom breakpoint-specific styling paths.

Impact:
- No immediate action required; implementation is intentional per developer clarification

Recommendation:
Keep as accepted unless product/design requests strict mobile/desktop style parity for age restriction modal.

**PR comment (line 35):**  
Accepted for now: remaining desktop-only styling in `Agerestrictionmodal` is intentional based on Tailwind breakpoint usage.
---

---
Connect Social mobile breakpoint widened (640px → md)

Risk Level: MEDIUM
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx
Lines: 60, 289

Description:
Accepted for now. `ConnectSocialModal` uses `useResponsiveModalLayout()` (`down("md")`), which broadens mobile behavior to ~768px. Previous 640px constant was removed.

Impact:
- Tablet widths (641–768) use mobile drawer behavior and always-revealed disconnect affordance

Recommendation:
Confirm with design/product; if intentional, keep accepted. If not intentional, use a dedicated 640px query for this modal’s interaction logic.

**PR comment (line 60):**  
Accepted for now: breakpoint behavior changed from 640px to `md`.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `modalClassNames` / `modalStyles` desktop-only | MEDIUM | Accepted | `Agerestrictionmodal.tsx`, `DistributeModal.tsx` | 35-38, 349-367 |
| 2 | Connect Social breakpoint 640 → md | MEDIUM | Accepted | `ConnectSocialModal.tsx` | 60, 289 |

**Fixed in latest re-review:**
- No-op drawer close on non-dismissable modals (`AthleteCreatedSuccessModal` now sets `showDrawerCloseButton={false}`)

**New issues introduced in latest fix commits:** None at Critical/High/Medium level.
