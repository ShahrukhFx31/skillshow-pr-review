# Frontend PR Review — skillshow-admin-ui (`SKSH-357`)

**Repo:** skillshow-admin-ui — `git@github-work:SkillshowFx/skillshow-admin-ui.git`  
**Branch:** `SKSH-357`  
**Base:** `main...HEAD` @ `d79e38c`  
**Initial review:** 2026-06-22  
**Re-review:** 2026-06-22  
**Scope:** Multi-country profile country selection, country-aware state/city selector, basic/sport profile layout refactor, `resolveSportRules` DRY, `lockDateOfBirth` (Critical / High / Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 1 (0 Critical, 1 High) — **0 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-server-table-controls.ts`, `use-pagination.ts`, audit-log components, `antd.adapter.tsx` | **Not modified** ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/account-general.constants.ts` | `ACCOUNT_COUNTRY_OPTIONS`, country helpers |
| `src/api/services/stateService.ts` | `getAllStates(countryCode?)`, `getCitiesByState(stateKey, countryCode?)` |
| `src/hooks/use-state-and-city-selector.ts` | Country in state/city queries; country-change reset |
| `src/pages/user/account/general/components/BasicInformationCard.tsx` | Country-aware read view; `lockDateOfBirth` prop |
| `src/pages/user/account/general/components/BasicInformationEditForm.tsx` | Editable country; country-scoped selector |
| `src/pages/user/account/general/index.tsx` | `lockDateOfBirth = !isCrew` |
| `src/pages/user/account/general/hooks/useAccountGeneralProfileEditor.ts` | `lockDateOfBirth` support |
| `src/pages/user/account/general/schemas/profile-forms.schema.ts` | `lockDateOfBirth` schema option |
| Other sport/profile/display files | Layout refactor + `resolveSportRules` |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Country helpers centralized in `account-general.constants.ts` | ✅ DRY |
| `resolveSportRules` shared across sport profile surfaces | ✅ DRY |
| Edit + read basic info use `accountCountryCodeFromValue` for state lists | ✅ Global consistency |
| `useStateAndCitySelector` passes `countryCode` to `getAllStates` and `getCitiesByState` | ✅ Contract alignment with backend |
| `StateCityFormFields` without `countryCode` | ✅ Accepted — onboarding country still disabled/US-only |
| `lockDateOfBirth` UI-only (payload strip); no backend mirror for non-minor self-edit | ✅ Accepted — minors already stripped server-side; adult DOB lock is product/UI policy in this PR; backend hardening can be follow-up |

### Re-review evidence (finding #1)

Commit `9cda3d79` updates `BasicInformationCard` to derive `countryCode` from `form.watch("country")`, removes `/^[A-Z]{2}$/` gate, and uses `getAllStates(countryCode)` in read view. `use-state-and-city-selector.ts` passes `countryCode` to `getCitiesByState` with matching `queryKey`.

### Positive notes

- Read and edit modes now share the same country → states API contract.
- Japan numeric prefecture codes (e.g. `13`) can resolve in read view once states are loaded.
- Linked-athlete editor (`useLinkedAthleteProfileEditor`) does not pass `lockDateOfBirth` — parents can still edit child DOB.
- `lockDateOfBirth` separates from `lockMinorAthleteDobAndGrad` (grad-only for minors).

---

## GitHub comments

_No open Critical or High findings._

**Resolved (was open):** Read-only `BasicInformationCard` country scoping — fixed in `9cda3d79` (lines 38–46).

---

## Findings

---
Read-only basic info card ignores profile country for state name resolution

Risk Level: HIGH  
File Path: src/pages/user/account/general/components/BasicInformationCard.tsx  
Lines: 38-46

Description:
**Global consistency** — Original review: read view hardcoded US states and `/^[A-Z]{2}$/` gate. Re-review: uses `accountCountryCodeFromValue(readOnlyCountry)`, `shouldLoadStateMap` on any non-empty state, and `getAllStates(countryCode)` / `queryKey: ["states", "all", countryCode]`.

Impact:
- Was: CA/JP profiles showed raw `state_id` in read mode.
- Now: resolved; aligns with edit form and backend API.

Recommendation:
N/A — implemented.

**Re-review:** ✅ Fixed @ `9cda3d79` / `d79e38c`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Read-only basic info card ignores profile country for state name resolution | HIGH | ✅ Fixed | src/pages/user/account/general/components/BasicInformationCard.tsx | 38-46 |

**Merge readiness:** ✅ No Critical or High blockers on the frontend. Country-aware state/city flows are consistent across read/edit UI and backend contract.
