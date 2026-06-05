# Frontend PR Review — skillshow-admin-ui (`SKSH-317`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-317`  
**Base:** `main...HEAD`  
**HEAD:** `ee737024` — Merge main + `efc7a0fa` refactor (hook/utils lift) + `5f25dd7f` initial  
**Reviewed:** 2026-06-05 — full `/pr-review` pass (entire diff)  
**Scope reviewed:** Full PR diff — **19 files** in `git diff main...HEAD`  
**Findings:** 0 Critical, 0 High, 1 Medium — **0 Open**, **1 Accepted**, **2 Fixed**  
**Note:** Review-only — findings for the developer; no agent code changes.

### Files reviewed

| File | Change |
|------|--------|
| `src/components/forms/StateCityFormFields.tsx` | Shared Ant Form state/city fields |
| `src/components/forms/index.ts` | Barrel export |
| `src/constants/state-city-selector.constants.ts` | `cityOptionsLoadingRow` promoted |
| `src/hooks/index.ts` | Export shared hook |
| `src/hooks/use-state-and-city-selector.ts` | Hook moved from account-general |
| `src/types/state-city-selector.types.ts` | Shared types |
| `src/utils/state-city-selector.utils.ts` | Shared utils |
| `src/pages/dashboard/crew/onboarding/form-helpers.tsx` | Inline state/city removed |
| `src/pages/dashboard/crew/onboarding/steps/background-tax-step.tsx` | `StateCityFormFields` direct |
| `src/pages/events/dashboard/components/events-bulk-update.tsx` | Removed eager `states` query |
| `src/pages/events/onboarding/components/event-form.tsx` | Dropped `states` / `statesLoading` |
| `src/pages/events/onboarding/components/schedule-location-section.tsx` | `StateCityFormFields` |
| `src/pages/events/onboarding/index.tsx` | Removed eager `states` query |
| `src/pages/events/onboarding/types.ts` | Prop cleanup |
| `src/pages/management/app-users/onboarding/components/app-user-form.tsx` | `StateCityFormFields` (inline Select removed) |
| `src/pages/user/account/general/components/StateAndCitySelector.tsx` | Hook import → `@/hooks/` |
| `src/pages/user/account/general/constants/index.ts` | `cityOptionsLoadingRow` removed |
| `src/pages/user/account/general/types/index.ts` | Selector types removed (moved up) |
| `src/pages/user/account/general/utils/index.ts` | State/city utils removed (moved up) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Shared hook lifted (`use-state-and-city-selector` → `src/hooks/`) | ✅ Fixed — see #2 |
| Shared utils/types/constants promoted (`src/utils/`, `src/types/`, `src/constants/`) | ✅ |
| All **PR-touched** Ant Form state/city UIs use `StateCityFormFields` | ✅ Crew, events (create/edit + bulk), app-users |
| `StateAndCitySelector` (RHF) uses shared hook from `@/hooks/` | ✅ |
| No pass-through wrappers | ✅ Fixed — see #3 |
| Eager `useQuery(["states", "all"])` removed from event entry points | ✅ |
| Legacy `states` / `statesLoading` prop chain removed end-to-end | ✅ |
| State-before-city field order on event form | ✅ **Accepted** — intentional |

### Positive notes

- **DRY / Reusability:** Single hook + utils + types power Ant Form (`StateCityFormFields`), RHF (`StateAndCitySelector`), crew, events, and app-users — ~321 lines removed net.
- **Global consistency:** `app-user-form.tsx` migrated in the same PR (no longer out-of-scope duplicate).
- **KISS:** `cityFirst` prop dropped; state-first is the one supported order (matches product intent).
- `notFoundContent`, `showSearch={{ optionFilterProp: "label" }}` added on shared Ant Selects.
- Lazy state/city loading; shared React Query cache (`["states", "all"]`).

---

---
Event form city/state column order changed (state first)

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: src/pages/events/onboarding/components/schedule-location-section.tsx  
Lines: 45

**Description:**  
On `main`, `ScheduleLocationSection` rendered **City** then **State**. This PR uses `StateCityFormFields` with state-first order. Developer confirmed this is **intentional** — aligns with crew onboarding and city-dependent-on-state UX.

**Impact:**  
Layout differs from legacy event form; acceptable product decision.

**Resolution:** Accepted — no change required.

---

---
Shared component promoted without lifting its hook (incomplete global refactor)

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/hooks/use-state-and-city-selector.ts (was `src/components/forms/StateCityFormFields.tsx` line 5)  
Lines: 1-17

**Description:**  
**DRY** + **Global consistency** — Prior review: `StateCityFormFields` imported hook from `pages/user/account/general`.

**Re-verification (`efc7a0fa` / `ee737024`):** ✅ **Fixed** — Hook renamed/moved to `src/hooks/use-state-and-city-selector.ts`; utils → `src/utils/state-city-selector.utils.ts`; types → `src/types/state-city-selector.types.ts`; constants → `src/constants/state-city-selector.constants.ts`. `StateCityFormFields` and `StateAndCitySelector` import from `@/hooks/`. Exported via `src/hooks/index.ts`. Account-general duplicates removed.

**PR comment (`StateCityFormFields.tsx` line 5):**  
**Resolved** — Hook and helpers are now shared under `src/hooks/`, `src/utils/`, `src/types/`, and `src/constants/`. Good complete extraction.

---

---
`CrewStateCityFields` is a no-op pass-through wrapper

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/dashboard/crew/onboarding/form-helpers.tsx  
Lines: (removed)

**Description:**  
**KISS** — Prior review: one-line wrapper relayed `form` to `StateCityFormFields`.

**Re-verification (`efc7a0fa` / `ee737024`):** ✅ **Fixed** — `CrewStateCityFields` removed from `form-helpers.tsx`; `background-tax-step.tsx` line 65 uses `<StateCityFormFields form={form} />` directly.

**PR comment (`background-tax-step.tsx` line 65):**  
**Resolved** — Direct `@/components/forms` import; wrapper removed.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Event form city/state column order changed (state first) | MEDIUM | Accepted | `schedule-location-section.tsx` | 45 |
| 2 | Shared component promoted without lifting its hook | HIGH | ✅ Fixed | `use-state-and-city-selector.ts` | 1-17 |
| 3 | `CrewStateCityFields` is a no-op pass-through wrapper | MEDIUM | ✅ Fixed | `form-helpers.tsx` / `background-tax-step.tsx` | 65 |

## GitHub comments (paste-ready)

**#1 — Accepted (no comment needed)**  
State-first order is intentional; dismiss any prior city-before-state feedback.

**#2 — Resolved (`StateCityFormFields.tsx` line 5)**  
Hook and helpers are now shared under `src/hooks/`, `src/utils/`, `src/types/`, and `src/constants/`. Complete extraction — looks good.

**#3 — Resolved (`background-tax-step.tsx` line 65)**  
`CrewStateCityFields` wrapper removed; call site uses `StateCityFormFields` directly from `@/components/forms`.

---

**Merge readiness (skillshow-admin-ui):** No open Critical/High/Medium blockers. All actionable findings fixed; field order (#1) accepted as intentional. **Ready to merge** from a frontend review perspective.
