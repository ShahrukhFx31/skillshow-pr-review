# Frontend PR Review — skillshow-admin-ui (`SKSH-264`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-264`  
**Base:** `main...HEAD` (46 files, ~1783 insertions / ~578 deletions)  
**Re-verified:** 2026-06-08 @ `a66e35db` (includes `4c2051be` shared sections, `03bb2e35` cache sync, `a66e35db` editor profile)  
**Scope:** Crew account profile, onboarding/profile shared sections, Performance & Reviews tab, sport position selects, header completion %, section-specific PATCH toasts (Critical, High, Medium)  
**Findings:** 5 (0 Critical, 2 High, 1 Medium) — **1 Accepted**, **4 Fixed**

---

---
Performance & Reviews tab renders hardcoded mock feedback

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx  
Lines: 5-12, 71-77

**Accepted:** Intentional UI placeholder for this release — static `CREW_PERFORMANCE_RATING_SUMMARY` / `CREW_PERFORMANCE_FEEDBACK_ROWS` until crew feedback API is available in a follow-up ticket. Tab also shown to Editor role (`account/index.tsx` `showPerformanceReviews = isCrew || isEditor`) with the same placeholder.

Description:
The **Performance & Reviews** account tab renders `CrewPerformanceReviews` from static constants (`crew/constants.ts`). No API call yet.

Impact:
- Placeholder data only; real ratings/feedback deferred to a later API integration.

Recommendation:
Replace mock constants with crew/editor feedback API when backend endpoint is ready (follow-up ticket).

**PR comment:** N/A — accepted as intentional placeholder.

---

---
Account-general and crew-onboarding caches diverge for shared user fields

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/utils/crew-account-cache-sync.ts  
Lines: 27-59

**Re-verification:** `03bb2e35` adds bidirectional sync; `general/index.tsx` calls `syncCrewOnboardingCacheFromAccountGeneral` from shared `mutation.onSuccess` when `isCrew` (lines 112–114); `useSaveCrewOnboardingStep` calls `syncAccountGeneralCacheFromCrewOnboarding` (`hooks.ts` 40–43).

Description:
Previously, basic/contact saves updated only `account-general` while profile sections and header completion read `CREW_ONBOARDING_QUERY_KEY`.

Impact:
- Resolved: overlapping name/phone/address fields merge across both caches.

Recommendation:
N/A — implemented.

---

---
Profile section cards duplicate onboarding step forms (DRY / Global consistency)

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/pages/dashboard/crew/shared/sections/  
Lines: (all section modules)

**Re-verification:** `4c2051be` moves sections to `pages/dashboard/crew/shared/sections/` and migrates all four onboarding steps to import shared `*EditFields`, `*ReadView`, and payload mappers (e.g. `background-tax-step.tsx` lines 4–8, 39). Profile cards import the same modules (e.g. `CrewBackgroundTaxCard.tsx` lines 4–9, 51–54).

Description:
Previously profile cards and onboarding wizard maintained parallel Ant form definitions.

Impact:
- Single source of truth for crew field UI, validation rules, and PATCH payloads across wizard and account profile.

Recommendation:
N/A — implemented.

---

---
`description` prop on `CrewProfileSectionCard` is never rendered

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/user/account/general/types/crew.types.ts  
Lines: 4-14

**Re-verification:** `description` removed from `CrewProfileSectionCardProps`; callers no longer pass a dead prop (`03bb2e35`).

Description:
Previously optional `description` was passed but never rendered.

Impact:
- Dead prop eliminated. Helper copy remains on onboarding via `OnboardingSection` descriptions (e.g. `background-tax-step.tsx` line 36).

Recommendation:
N/A — resolved by removal.

---

---
Duplicate mobile/desktop edit buttons in section card header

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/user/account/general/crew/components/CrewProfileSectionCard.tsx  
Lines: 30-40

**Re-verification:** Collapsed to a single edit `Button` (`03bb2e35`).

Description:
Previously two identical edit buttons differed only by responsive visibility classes.

Impact:
- Resolved.

Recommendation:
N/A.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Performance & Reviews tab renders hardcoded mock feedback | HIGH | Accepted | src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx | 5-12, 71-77 |
| 2 | Account-general and crew-onboarding caches diverge for shared user fields | HIGH | ✅ Fixed | src/utils/crew-account-cache-sync.ts | 27-59 |
| 3 | Profile section cards duplicate onboarding step forms | HIGH | ✅ Fixed | src/pages/dashboard/crew/shared/sections/ | all |
| 4 | `description` prop on `CrewProfileSectionCard` is never rendered | MEDIUM | ✅ Fixed | src/pages/user/account/general/types/crew.types.ts | 4-14 |
| 5 | Duplicate mobile/desktop edit buttons in section card header | MEDIUM | ✅ Fixed | src/pages/user/account/general/crew/components/CrewProfileSectionCard.tsx | 30-40 |

**Positive notes:** Shared crew sections under `dashboard/crew/shared/sections/` consumed by both onboarding wizard and profile cards; `crew-account-cache-sync.ts` bridges overlapping query caches; section-specific PATCH toasts align with backend; sport position selects share `sport-field-options.ts`; crew header completion uses dedicated crew query; `a66e35db` adds Editor admin-like profile layout and Performance & Reviews tab visibility.

**Merge readiness:** **Merge-ready** — all findings Fixed or Accepted; no open Critical/High/Medium blockers. Backend and frontend cleared for merge.
