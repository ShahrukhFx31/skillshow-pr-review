# Frontend PR Review — skillshow-admin-ui (`SKSH-264`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-264`  
**Base:** `main...HEAD` (29 files, ~1539 insertions / ~185 deletions)  
**Scope:** Crew account profile (onboarding sections on Profile tab), Performance & Reviews tab, sport position selects, header completion %, section-specific PATCH toasts (Critical, High, Medium)  
**Findings:** 5 (0 Critical, 3 High, 2 Medium)

---

---
Performance & Reviews tab renders hardcoded mock feedback

Risk Level: HIGH
File Path: src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx
Lines: 1-85

Description:
The new **Performance & Reviews** account tab (`performance-reviews-tab.tsx`) renders `CrewPerformanceReviews`, which reads entirely from static constants in `src/pages/user/account/general/crew/constants.ts` (`CREW_PERFORMANCE_RATING_SUMMARY`, `CREW_PERFORMANCE_FEEDBACK_ROWS`). There is no API call, loading state, or empty-state guard. The UI presents fictional ratings (e.g. 4.8 average, 12 reviews, named athletes/events) as live data.

Impact:
- Crew users see fabricated performance metrics and client feedback in a production account tab.
- Misleading ratings can affect trust and operational decisions; no path to real edit-request / client feedback data exists in this PR.

Recommendation:
Do not ship the tab with placeholder rows. Either wire to the real crew feedback/ratings API (e.g. dashboard KPI / edit-request feedback endpoints when available), or hide the tab behind a feature flag and show an explicit empty/coming-soon state until the API exists. Remove `CREW_PERFORMANCE_*` mock exports before release.

**PR comment (line 9):**  
**High:** This tab currently renders static mock ratings and feedback (`CREW_PERFORMANCE_RATING_SUMMARY` / `CREW_PERFORMANCE_FEEDBACK_ROWS`) with no API — please wire real data or gate the tab until backend support exists; shipping fictional reviews is misleading for crew users.

---

---
Account-general and crew-onboarding caches diverge for shared user fields

Risk Level: HIGH
File Path: src/pages/user/account/general/index.tsx (also `src/pages/dashboard/crew/hooks.ts`, `src/layouts/components/account-dropdown.tsx`)
Lines: 282, 730 (`index.tsx`); 39 (`hooks.ts`); 60–70 (`account-dropdown.tsx`)

Description:
Crew profile completion (`computeCrewProfileCompletionPercentage` in `account-dropdown.tsx`) and the onboarding section cards use `CREW_ONBOARDING_QUERY_KEY`, while **Basic Information** and **Contact Information** saves go through `accountService.updateAccountGeneral` and only update `PROFILE_QUERY_KEYS.accountGeneral` (pre-existing `setQueryData` at `index.tsx` 107–108 / 342–343 — **unchanged in this PR, not commentable on GitHub**). Conversely, `useSaveCrewOnboardingStep` updates `CREW_ONBOARDING_QUERY_KEY` only. Phone, first name, and last name are the same underlying user fields (`user.phoneNumber`, `user.firstName`, `user.lastName` per `crew-onboarding.service.ts`) but are editable from both stacks without cross-cache sync.

Impact:
- Updating phone in Contact leaves Background & Tax read view and header completion % stale until a full refetch.
- Updating phone in Background & Tax leaves Contact read view stale.
- Saving first/last name in Basic Information does not refresh crew onboarding cache, so header completion % can stay below 100% until reload.

Recommendation:
On crew role, cross-invalidate or merge caches after either mutation path, e.g. in `useSaveCrewOnboardingStep.onSuccess` invalidate/set `PROFILE_QUERY_KEYS.accountGeneral`, and in `handleSaveBasic` / `handleSaveContact` when `isCrew` invalidate or patch `CREW_ONBOARDING_QUERY_KEY` with the overlapping fields returned from the PUT response. Alternatively, route crew name/phone edits exclusively through `patchCrewOnboarding` so one query key remains authoritative.

**PR comment — use a changed line in the diff (GitHub cannot comment on unchanged lines 107/342):**

| File | Line (branch) | Why it's in the PR diff |
|------|---------------|-------------------------|
| `src/pages/user/account/general/index.tsx` | **282** | New crew basic save sends only `firstName`/`lastName` via `updateAccountGeneral` |
| `src/pages/user/account/general/index.tsx` | **730** | New `CrewOnboardingProfileSections` reads `CREW_ONBOARDING_QUERY_KEY` |
| `src/pages/dashboard/crew/hooks.ts` | **39** | Crew PATCH `onSuccess` updates only `CREW_ONBOARDING_QUERY_KEY` |
| `src/layouts/components/account-dropdown.tsx` | **70** | Header completion % now reads `crewProfile`, not `account-general` |

**Suggested inline comment (`index.tsx` line 282):**  
**High:** Crew basic info now saves via `updateAccountGeneral` only, but completion % (`account-dropdown.tsx`) and profile sections (`CrewOnboardingProfileSections`) read `CREW_ONBOARDING_QUERY_KEY` — shared phone/name can stay stale until reload. Cross-sync or invalidate both caches on crew saves (same gap for contact + `hooks.ts:39` on the reverse path).

---

---
Profile section cards duplicate onboarding step forms (DRY / Global consistency)

Risk Level: HIGH
File Path: src/pages/user/account/general/crew/components/cards/
Lines: 1-672 (all four card files)

Description:
**DRY / Global consistency:** The four new profile cards (`CrewBackgroundTaxCard`, `CrewJobPreferencesCard`, `CrewFilmingEditingCard`, `CrewEquipmentCard`) largely copy the Ant Design forms, validation rules, read views, and save payloads already implemented under `src/pages/dashboard/crew/onboarding/steps/`. This PR adds ~650 lines of parallel UI for the same fields instead of extracting shared step components or a reusable “crew section form” module consumed by both onboarding routes and account profile.

Impact:
- Validation, field lists, and save mapping can drift between onboarding wizard and account profile (e.g. future rule changes applied in one path only).
- Double maintenance cost for every crew field change.

Recommendation:
Extract shared presentational + form modules from the onboarding steps (e.g. `BackgroundTaxFields`, `JobPreferencesFields`) and compose them in both `onboarding/steps/*` and `general/crew/components/cards/*`. Keep section titles / toast labels in one constants module (`CREW_PROFILE_SECTION_TITLES` already exists).

**PR comment (`CrewBackgroundTaxCard.tsx` line 21 — new file, comment on file or `useSaveCrewOnboardingStep` at line 25):**  
**High (DRY):** These profile cards mirror the onboarding step forms almost line-for-line — consider extracting shared field/read/edit components used by both `/dashboard/crew/onboarding` and account profile so validation and PATCH payloads stay in one place.

---

---
`description` prop on `CrewProfileSectionCard` is never rendered

Risk Level: MEDIUM
File Path: src/pages/user/account/general/crew/components/CrewProfileSectionCard.tsx
Lines: 7-17, 25-28

Description:
`CrewProfileSectionCardProps` includes optional `description`, and callers such as `CrewBackgroundTaxCard` and `CrewJobPreferencesCard` pass it, but the component destructures only `title` and never renders `description`. Onboarding steps show equivalent copy via `OnboardingSection`.

Impact:
- Profile cards lose helper text that exists in the onboarding flow (e.g. “Please provide your primary residence…”), inconsistent UX between first-time onboarding and profile editing.

Recommendation:
Render `description` under the title when provided (match onboarding subtitle styling), or remove the prop from the type and call sites.

**PR comment (line 27):**  
**Medium:** Cards pass `description` but `CrewProfileSectionCard` never renders it — crew profile sections are missing the helper copy shown during onboarding.

---

---
Duplicate mobile/desktop edit buttons in section card header

Risk Level: MEDIUM
File Path: src/pages/user/account/general/crew/components/CrewProfileSectionCard.tsx
Lines: 30-51

Description:
**DRY:** Two identical `Button` elements differ only by `sm:hidden` vs `hidden sm:inline-flex`. Same markup, handlers, and classes are duplicated.

Impact:
- Future header changes must be applied twice; easy to miss one breakpoint variant.

Recommendation:
Use a single `Button` with responsive classes, or a tiny `EditIconButton` helper shared with athlete profile cards if they use the same pattern.

**PR comment (line 32):**  
**Medium (DRY):** The edit control is duplicated for mobile/desktop — one button with responsive visibility classes would be simpler to maintain.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Performance & Reviews tab renders hardcoded mock feedback | HIGH | Open | src/pages/user/account/general/crew/components/CrewPerformanceReviews.tsx | 1-85 |
| 2 | Account-general and crew-onboarding caches diverge for shared user fields | HIGH | Open | src/pages/user/account/general/index.tsx | 282, 730 (+ hooks.ts 39) |
| 3 | Profile section cards duplicate onboarding step forms | HIGH | Open | src/pages/user/account/general/crew/components/cards/ | 1-672 |
| 4 | `description` prop on `CrewProfileSectionCard` is never rendered | MEDIUM | Open | src/pages/user/account/general/crew/components/CrewProfileSectionCard.tsx | 7-28 |
| 5 | Duplicate mobile/desktop edit buttons in section card header | MEDIUM | Open | src/pages/user/account/general/crew/components/CrewProfileSectionCard.tsx | 30-51 |

**Positive notes:** Section-specific PATCH toasts (`patchCrewOnboarding` + `CREW_PROFILE_SECTION_TITLES`) align with the backend query param; crew header completion uses `computeCrewProfileCompletionPercentage` with a dedicated crew query; sport position/bat/throw selects share `sport-field-options.ts` with zod validation in `createSportProfileFormSchema`; athlete profile cards gain reusable `nameOnly` / `showPreferredContactMethod` flags without breaking existing flows.

**Merge readiness:** **Not merge-ready** — 3 open High findings (mock performance data, split query caches, duplicated onboarding forms). Address or explicitly accept before merge.
