# Frontend PR Review — skillshow-admin-ui (`SKSH-168`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-168`  
**Base:** `main...HEAD`  
**Scope:** Connect Social unlink confirmation modal (Critical, High, Medium)  
**Commits:** `7345e861` fix: 168 · `e5c5e537` fix: feedbacks  
**Files changed:** 1 (`ConnectSocialModal.tsx`)  
**Re-review:** 2026-06-02 — all prior findings verified fixed on branch

---

---
Destructive action appears above Cancel on mobile

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx  
Lines: 558-579

Description:
The confirm footer switched from `vertical={isMobile}` (natural column order) to `flex-col-reverse` with `flex-col-reverse items-stretch md:flex-row`. Below the `md` breakpoint, `flex-col-reverse` rendered the second button (Unlink) above Cancel.

Impact:
- On drawer/mobile layouts, the primary destructive control was the first tap target, increasing accidental unlink risk.
- Behavior diverged from the previous mobile layout without an obvious product reason.

Recommendation:
Drop `flex-col-reverse` so DOM order matches visual order on small screens (Cancel on top, Unlink below).

**Re-review:** ✅ Fixed in `e5c5e537` — footer uses `flex-col items-stretch md:flex-row md:items-center` (no `flex-col-reverse`); Cancel remains first in DOM on mobile.

---

---
Mixed “Disconnect” and “Unlink” copy in the same dialog

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx  
Lines: 375, 550-579

Description:
The confirmation modal title used “Disconnect” while the primary button label was “Unlink”. The list-row control exposed `aria-label="Disconnect"`.

Impact:
- Inconsistent terminology in one flow (title vs CTA vs screen reader label).
- Users and assistive tech could hear “Disconnect” while the visible action said “Unlink”.

Recommendation:
Align copy to one term (“Unlink”).

**Re-review:** ✅ Fixed in `e5c5e537` — title `Unlink ${confirmProvider.name} Account?` / `Unlink Account?`, CTA `Unlink`, list control `aria-label="Unlink"`.

---

---
`disconnectWarning` has no fallback when provider lookup fails

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx  
Lines: 164-170, 554-555

Description:
Warning copy depended on `confirmProvider?.name` with no fallback when provider lookup failed.

Impact:
- Confirm modal could show a title with no explanatory body copy in edge cases.

Recommendation:
Restore a generic fallback when `platformName` is missing.

**Re-review:** ✅ Fixed in `e5c5e537` — generic message when `!platformName`: “You won't be able to share photos or videos to this platform until it's connected again.”

---

## Positive notes

- Dynamic warning from `confirmProvider.name` matches product copy (“share photos or videos”) and stays correct for “X” vs legacy ids.
- `useMemo` dependency on `confirmProvider` is appropriate after dropping the `switch`.
- Full-width footer buttons via Tailwind (`w-full!` / `md:w-auto!`) and `[&_.ant-space-item]:w-full` improve layout without extra `block={isMobile}` props.
- `md:` breakpoint aligns with `useResponsiveModalLayout()` (`down("md")`).
- Feedback commit addressed all three review items without regressions.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Destructive action above Cancel on mobile | MEDIUM | ✅ Fixed | src/pages/dashboard/components/ConnectSocialModal.tsx | 558-579 |
| 2 | Mixed “Disconnect” / “Unlink” copy | MEDIUM | ✅ Fixed | src/pages/dashboard/components/ConnectSocialModal.tsx | 375, 550-579 |
| 3 | No fallback when `confirmProvider` is missing | MEDIUM | ✅ Fixed | src/pages/dashboard/components/ConnectSocialModal.tsx | 164-170, 554-555 |

**Merge readiness:** No open Critical/High/Medium blockers. All findings resolved on branch — ready to merge from a frontend review perspective.
