# Frontend PR Review — skillshow-admin-ui (`SKSH-357`)

**Repo:** skillshow-admin-ui — `git@github-work:SkillshowFx/skillshow-admin-ui.git`  
**Branch:** `SKSH-357`  
**Base:** `main...HEAD` @ `c8bbd76`  
**Initial review:** 2026-06-22  
**Scope:** Multi-country profile country selection, country-aware state/city selector, basic/sport profile layout refactor, `resolveSportRules` DRY (Critical / High / Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 1 (0 Critical, 1 High) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-server-table-controls.ts`, `use-pagination.ts`, audit-log components, `antd.adapter.tsx` | **Not modified** ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/account-general.constants.ts` | `ACCOUNT_COUNTRY_OPTIONS`, helpers (`accountCountryCodeFromValue`, `normalizeAccountCountryValue`, etc.) |
| `src/api/services/stateService.ts` | `getAllStates(countryCode?)` passes `?country=` |
| `src/hooks/use-state-and-city-selector.ts` | `countryCode` param, query key, country-change reset, city sort |
| `src/pages/user/account/general/components/BasicInformationEditForm.tsx` | Editable country; country-scoped `StateAndCitySelector`; two-column layout |
| `src/pages/user/account/general/components/BasicInformationCard.tsx` | Two-column read layout; `accountCountryDisplayLabel` |
| `src/pages/user/account/general/components/StateAndCitySelector.tsx` | `countryCode`, `cityFirst` props; extracted field blocks |
| `src/pages/user/account/general/constants/index.ts` | Left/right column field arrays; `getBasicInfoColumnFields` |
| `src/pages/user/account/general/constants/sport-field-options.ts` | `resolveSportRules` helper; soccer/volleyball secondary position |
| `src/pages/user/account/general/schemas/profile-forms.schema.ts` | `z.enum(ACCOUNT_COUNTRY_VALUES)` |
| `src/pages/user/account/general/utils/basic-info-form.ts` | Hydrate country from API via `normalizeAccountCountryValue` |
| `src/utils/state-city-selector.utils.ts` | Alphabetical city option sort |
| `src/pages/management/app-users/onboarding/components/app-user-form.tsx` | `ACCOUNT_COUNTRY_OPTIONS` (country field still disabled) |
| Other sport/profile/display files | Layout refactor + `resolveSportRules` adoption |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Country display/normalization centralized in `account-general.constants.ts` | ✅ DRY |
| `resolveSportRules` replaces repeated sport-rule lookup across 6+ files | ✅ DRY |
| `useStateAndCitySelector` + `stateService.getAllStates(countryCode)` wired in edit form | ✅ |
| `BasicInformationCard` read view still hardcodes US states fetch | ❌ **Global consistency** — see finding #1 |
| `StateCityFormFields` (shared Ant form wrapper) not updated with `countryCode` | ✅ Accepted — app-user onboarding country remains disabled/US-only; account profile uses `StateAndCitySelector` directly |
| `cityFirst` prop added but unused in this diff | ✅ Accepted — optional layout hook for future consumers |
| `getCitiesByState` not passed `countryCode` in hook | ⚠️ Blocked on backend contract (see `backend.md` #1) |

### Positive notes

- Country change in edit form clears state/city and re-fetches states via updated `queryKey` — correct UX.
- Frontend `ACCOUNT_COUNTRY_VALUES` aligns with backend `PROFILE_ACCOUNT_COUNTRIES`.
- Legacy `USA` stored value normalized on hydrate via `findAccountCountryByValue`.
- Sport profile two-column layout matches basic info column pattern; `resolveSportRules` reduces drift.
- City options sorted alphabetically in both hook and utils for consistent dropdown order.

---

## GitHub comments

### `src/pages/user/account/general/components/BasicInformationCard.tsx` (≈37–44)

The edit form correctly passes `accountCountryCodeFromValue(selectedCountry)` into `StateAndCitySelector`, but the read-only card still loads states with `getAllStates()` (implicit US) and `queryKey: ["states", "all", "US"]`. It also gates the fetch with `/^[A-Z]{2}$/`, which excludes Japan prefecture codes (e.g. `13`). For Canada/Japan profiles, the read view will show the raw `state_id` instead of the province/prefecture name. Please derive `countryCode` from `form.watch("country")` (same helper as edit) and relax the state-map gate to any non-empty stored state value.

---

## Findings

---
Read-only basic info card ignores profile country for state name resolution

Risk Level: HIGH  
File Path: src/pages/user/account/general/components/BasicInformationCard.tsx  
Lines: 37-44

Description:
**Global consistency** — This PR migrates the edit flow to country-aware state loading (`accountCountryCodeFromValue` → `StateAndCitySelector` → `getAllStates(countryCode)`), but the read-only view in `BasicInformationCard` still fetches states with a hardcoded US query (`getAllStates()` / `queryKey: ["states", "all", "US"]`). The `shouldLoadStateMap` guard uses `/^[A-Z]{2}$/`, which matches US/CA codes but excludes Japan numeric prefecture codes (documented on the backend as e.g. `13`). Canada and Japan users will see unresolved `state_id` values in read mode even when edit mode works.

Impact:
- Profile owners with `country: Canada` or `country: Japan` see abbreviated state codes in the collapsed/read basic-info card instead of full province/prefecture names.
- Inconsistent UX between read and edit modes on the same page.
- Undermines the multi-country feature for the primary account-general surface.

Recommendation:
```tsx
const readOnlyCountry = form.watch("country");
const countryCode = accountCountryCodeFromValue(readOnlyCountry);
const readOnlyState = (form.watch("state") ?? "").trim();
const shouldLoadStateMap = isOpen && !isEditing && !!readOnlyState;

const { data: states = [] } = useQuery({
  enabled: shouldLoadStateMap,
  queryFn: () => stateService.getAllStates(countryCode),
  queryKey: ["states", "all", countryCode],
});
```
Keep `getStateNameFromId` fallback for legacy full-name storage.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Read-only basic info card ignores profile country for state name resolution | HIGH | Open | src/pages/user/account/general/components/BasicInformationCard.tsx | 37-44 |

**Merge readiness:** ❌ One open High blocker on the frontend, plus backend cities contract (`backend.md` #1). Fix read-view country scoping and align `getCitiesByState` with `countryCode` before merging CA/JP support.
