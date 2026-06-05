# Frontend PR Review — skillshow-admin-ui (`SKSH-317`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-317`  
**Base:** `main...HEAD`  
**Commit:** `5f25dd7f` — feat: add StateCityFormFields component and integrate into event forms  
**Reviewed:** 2026-06-05 — full `/pr-review` pass (entire diff)  
**Scope reviewed:** Full PR diff — **8 files** in `git diff main...HEAD`  
**Findings:** 0 Critical, 1 High, 2 Medium — **2 Open**, **1 Accepted**  
**Note:** Findings are for the developer to address; review-only (no agent fixes).

### Files reviewed

| File | Change |
|------|--------|
| `src/components/forms/StateCityFormFields.tsx` | New shared Ant Form state/city fields |
| `src/components/forms/index.ts` | Barrel export |
| `src/pages/dashboard/crew/onboarding/form-helpers.tsx` | Deduped to `StateCityFormFields`; adds `CrewStateCityFields` wrapper |
| `src/pages/events/dashboard/components/events-bulk-update.tsx` | Removed eager `states` query |
| `src/pages/events/onboarding/components/event-form.tsx` | Dropped `states` / `statesLoading` props |
| `src/pages/events/onboarding/components/schedule-location-section.tsx` | Replaced city `Input` + state `Select` with `StateCityFormFields` |
| `src/pages/events/onboarding/index.tsx` | Removed eager `states` query |
| `src/pages/events/onboarding/types.ts` | Removed `states` / `statesLoading` from prop types |

### DRY / KISS / Global consistency scan

| Check | Verdict |
|-------|---------|
| All **PR-touched** Ant Form state/city UIs use `StateCityFormFields` | ✅ Crew (`form-helpers`) + events (`schedule-location-section` → create/edit + bulk) |
| Eager `useQuery(["states", "all"])` removed from all touched event entry points | ✅ `onboarding/index.tsx`, `events-bulk-update.tsx` |
| Legacy `states` / `statesLoading` prop chain removed end-to-end | ✅ `types.ts` → `event-form.tsx` → `schedule-location-section.tsx` |
| Shared hook lifted with shared component (`useStateAndCitySelector` → `src/hooks/` or similar) | ❌ **Open** — see #2 |
| No pass-through wrappers in PR diff | ❌ **Open** — see #3 |
| State-before-city field order on event form | ✅ **Accepted** — intentional; aligns with crew + `StateCityFormFields` default |

**Out of PR scope (not flagged):** `app-user-form.tsx` still inlines the same Ant Select + `useStateAndCitySelector` pattern — not in this diff; follow-up if Ant Form state/city should be unified repo-wide.

### Positive notes

- **DRY:** Crew and event forms share one implementation (~70 duplicated lines removed).
- **Global consistency:** Every state/city consumer **in this PR** migrated to `StateCityFormFields`; no mixed old/new patterns among touched files.
- Lazy state/city loading via `useStateAndCitySelector` removes mount-time fetches — shared React Query cache (`["states", "all"]`).
- `allowClear={bulkUpdate}` correctly keeps required fields non-clearable on create/edit while allowing partial bulk patches.
- Event city field moves from free-text `Input` to API-backed `Select` (consistent with crew); `buildCityOptionsForControl` preserves saved custom cities on edit.

---

---
Event form city/state column order changed (state first)

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: src/pages/events/onboarding/components/schedule-location-section.tsx  
Lines: 45

**Description:**  
On `main`, `ScheduleLocationSection` rendered **City** then **State**. This PR uses `StateCityFormFields` with the default state-first order (no `cityFirst`). Developer confirmed this is **intentional** — event forms should match crew onboarding and the shared component default (state → city), since city options depend on state selection.

**Impact:**  
Layout differs from the legacy event form; acceptable product decision.

**Resolution:** Accepted — no `cityFirst` needed. The `cityFirst` prop remains available for any future screen that requires city-before-state.

---

---
Shared component promoted without lifting its hook (incomplete global refactor)

Risk Level: HIGH  
**Status:** Open  
File Path: src/components/forms/StateCityFormFields.tsx  
Lines: 5, 40-46

**Description:**  
**DRY** + **Global consistency** — The PR promotes `StateCityFormFields` to `src/components/forms/` (cross-feature shared UI) but keeps `useStateAndCitySelector` under `@/pages/user/account/general/hooks/`. Shared UI now depends on a feature-page module (`utils`, `constants`, `types` under account-general). That is a partial migration: the component is global, its data orchestration is not. Re-verified on `5f25dd7f`: import path unchanged.

**Impact:**
- Refactors under `pages/user/account/general/` can break event and crew forms unexpectedly.
- Future Ant Form consumers (e.g. `app-user-form.tsx`, which already duplicates the same hook + Select pattern) will keep importing from account-general or copy-paste markup instead of reusing `StateCityFormFields`.
- Inverted dependency graph violates established layout (`src/components/` should not depend on `src/pages/<feature>/`).

**Recommendation:**  
Complete the global refactor in this PR (or split as immediate follow-up before merge):

1. Move `useStateAndCitySelector` + colocated utils/types/constants to `src/hooks/use-state-and-city-selector.ts` (or `src/components/forms/hooks/`).
2. Update imports in **PR files**: `StateCityFormFields.tsx`.
3. Re-export from account-general for backward compatibility if needed.

**PR comment (`StateCityFormFields.tsx` line 5):**  
**High (DRY / Global consistency):** `StateCityFormFields` is shared under `src/components/forms/` but still imports `useStateAndCitySelector` from `pages/user/account/general`. Please lift the hook (and helpers) to `src/hooks/` or `src/components/forms/hooks/` so the global extraction is complete and cross-feature forms do not depend on account-general internals.

---

---
`CrewStateCityFields` is a no-op pass-through wrapper

Risk Level: MEDIUM  
**Status:** Open  
File Path: src/pages/dashboard/crew/onboarding/form-helpers.tsx  
Lines: 6

**Description:**  
**KISS** — `CrewStateCityFields` only forwards `form` to `StateCityFormFields` with no crew-specific rules, layout, or behavior. The PR introduced this wrapper when deduplicating crew markup instead of using the shared component at the call site. Re-verified on `5f25dd7f`: unchanged one-liner relay.

**Impact:**
- Extra indirection — readers must open two files to understand crew state/city fields.
- Encourages feature-specific aliases for every shared form widget instead of importing `@/components/forms` directly.

**Recommendation:**  
Remove `CrewStateCityFields` from `form-helpers.tsx` and import `StateCityFormFields` directly in `background-tax-step.tsx` (only consumer). Keep `form-helpers.tsx` for helpers that add real behavior (`YesNoField`).

```tsx
// background-tax-step.tsx
import { StateCityFormFields } from "@/components/forms";
// ...
<StateCityFormFields form={form} />
```

**PR comment (`form-helpers.tsx` line 6):**  
**Medium (KISS):** This one-liner only relays `form` to `StateCityFormFields` with no added behavior. Consider removing `CrewStateCityFields` and importing `StateCityFormFields` directly from `@/components/forms` in `background-tax-step.tsx` (line 64).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Event form city/state column order changed (state first) | MEDIUM | Accepted | `schedule-location-section.tsx` | 45 |
| 2 | Shared component promoted without lifting its hook | HIGH | Open | `StateCityFormFields.tsx` | 5, 40-46 |
| 3 | `CrewStateCityFields` is a no-op pass-through wrapper | MEDIUM | Open | `form-helpers.tsx` | 6 |

**Merge readiness (skillshow-admin-ui):** One Open **High** (#2) plus one Open **Medium** (#3) for the developer. #1 Accepted as intentional. Review-only — no agent fixes applied.
