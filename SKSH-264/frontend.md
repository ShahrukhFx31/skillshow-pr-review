# Frontend PR Review — skillshow-admin-ui (`SKSH-264`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-264`  
**Base:** `main...HEAD` (41 files, ~1746 insertions / ~186 deletions)  
**Re-verified:** 2026-06-08 @ `4b5a5823` (includes `03bb2e35` fix: feedbacks)  
**Scope:** Crew account profile, Performance & Reviews tab, sport position selects, header completion %, section-specific PATCH toasts (Critical, High, Medium)  
**Findings:** 5 (0 Critical, 2 High, 1 Medium) — **1 Accepted**, **1 Partially fixed**, **3 Fixed**

---

---

Performance & Reviews tab renders hardcoded mock feedback

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx  
Lines: 5-12, 71-77

**Accepted:** Intentional UI placeholder for this release — static `CREW_PERFORMANCE_RATING_SUMMARY` / `CREW_PERFORMANCE_FEEDBACK_ROWS` ship until crew feedback API is available in a follow-up ticket.

Description:
The **Performance & Reviews** account tab renders `CrewPerformanceReviews` from static constants (`crew/constants.ts`). No API call yet.

Impact:
- Placeholder data only; real ratings/feedback deferred to a later API integration.

Recommendation:
Replace mock constants with crew feedback API when backend endpoint is ready (track in follow-up ticket).

**PR comment (`CrewPerformanceReviews.tsx` line 12):**  
N/A — accepted as intentional placeholder for this release.

---

---

Account-general and crew-onboarding caches diverge for shared user fields

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/utils/crew-account-cache-sync.ts  
Lines: 27-59

**Re-verification:** `03bb2e35` adds `syncCrewOnboardingCacheFromAccountGeneral` / `syncAccountGeneralCacheFromCrewOnboarding`. `general/index.tsx` calls crew sync from shared `mutation.onSuccess` when `isCrew` (lines 110–112). `useSaveCrewOnboardingStep` syncs account-general on crew PATCH success (`hooks.ts` 40–43).

Description:
Previously, basic/contact saves updated only `account-general` while profile sections and header completion read `CREW_ONBOARDING_QUERY_KEY`, and crew PATCH updated only the crew cache.

Impact:

- Resolved: name/phone/address overlap now merges across both caches (or invalidates crew cache when not yet loaded).

Recommendation:
N/A — implemented.

---

---

Profile section cards duplicate onboarding step forms (DRY / Global consistency)

Risk Level: HIGH  
**Status:** Partially fixed  
File Path: src/pages/user/account/general/crew/sections/  
Lines: (all section modules)

**Re-verification:** `03bb2e35` extracts shared `*EditFields`, `*ReadView`, and payload helpers; profile cards are thin wrappers (~~50–60 lines each). **Onboarding wizard** (`src/pages/dashboard/crew/onboarding/steps/`*) was **not** migrated — still inline Ant forms (~~100–200 lines per step).

Description:
**DRY / Global consistency:** Profile path now shares section modules under `general/crew/sections/`, but the PR leaves `dashboard/crew/onboarding/steps/`* on the legacy inline implementation. Field rules and save mapping can still drift between first-time onboarding and account profile editing.

Impact:

- Reduced duplication on profile cards; onboarding vs profile still maintain parallel form definitions.
- Future field changes may require edits in both `sections/*` and `onboarding/steps/*`.

Recommendation:
Migrate onboarding steps to import the same `sections/*` edit/read components and payload mappers (or move `sections/` to `pages/dashboard/crew/shared/` consumed by both routes).

**PR comment — use a line in the diff (GitHub cannot comment on unchanged `onboarding/steps/*`):**

| File | Line | Why commentable |
|------|------|-----------------|
| `src/pages/user/account/general/crew/components/cards/CrewBackgroundTaxCard.tsx` | **6** | New file; imports shared `BackgroundTaxEditFields` |
| `src/pages/user/account/general/crew/sections/background-tax/BackgroundTaxEditFields.tsx` | **14** | New file; shared field module onboarding should reuse |

**Suggested inline comment (`CrewBackgroundTaxCard.tsx` line 6):**  
**High (DRY, partial):** Profile cards now consume shared `sections/*` — please migrate `src/pages/dashboard/crew/onboarding/steps/*` (e.g. `background-tax-step.tsx`) to these same edit/read components and payload helpers so wizard and account profile stay one source of truth.

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

- Dead prop eliminated. Helper copy remains onboarding-only via `OnboardingSection` (acceptable tradeoff).

Recommendation:
N/A — resolved by removal. Optionally add subtitle to section cards later for parity with onboarding.

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


| #   | Title                                                                     | Risk   | Status          | File                                                                      | Lines       |
| --- | ------------------------------------------------------------------------- | ------ | --------------- | ------------------------------------------------------------------------- | ----------- |
| 1   | Performance & Reviews tab renders hardcoded mock feedback                 | HIGH   | Accepted        | src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx | 5-12, 71-77 |
| 2   | Account-general and crew-onboarding caches diverge for shared user fields | HIGH   | ✅ Fixed         | src/utils/crew-account-cache-sync.ts                                      | 27-59       |
| 3   | Profile section cards duplicate onboarding step forms                     | HIGH   | Partially fixed | src/pages/user/account/general/crew/sections/                             | all         |
| 4   | `description` prop on `CrewProfileSectionCard` is never rendered          | MEDIUM | ✅ Fixed         | src/pages/user/account/general/types/crew.types.ts                        | 4-14        |
| 5   | Duplicate mobile/desktop edit buttons in section card header              | MEDIUM | ✅ Fixed         | src/pages/user/account/general/crew/components/CrewProfileSectionCard.tsx | 30-40       |


**Positive notes:** `crew-account-cache-sync.ts` cleanly bridges overlapping fields both ways; profile cards are thin shells over shared section modules; section-specific PATCH toasts align with backend; sport position selects share `sport-field-options.ts`; crew header completion uses dedicated crew query.

**Merge readiness:** **Merge-ready with follow-up** — mock Performance & Reviews data **Accepted** (intentional placeholder). Remaining item: **Partially fixed** DRY — migrate `onboarding/steps/*` to shared `sections/*` in a follow-up. Backend ready; no open Critical/High blockers.