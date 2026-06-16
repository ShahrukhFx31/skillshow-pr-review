# Frontend PR Review — skillshow-admin-ui (`SKSH-349-1`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-349-1`  
**Base:** `main...HEAD` @ `193a9941`  
**Initial review:** 2026-06-16  
**Re-review:** 2026-06-16  
**Scope:** Event date/city validation parity, add-page header copy, import-tool empty-CSV guard, `StateCityFormFields` disabled/clear behavior (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 1 (0 Critical, 0 High, 1 Medium) — **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/events/constants.ts` | `EVENT_ADD_PAGE_*`, `EVENT_MAX_PAST_YEARS`, `eventDatePastRangeMessage` |
| `src/pages/events/utils/event-schedule-validation.utils.ts` | 25-year range helpers, form rules, `disabledDate` helper |
| `src/pages/events/onboarding/components/schedule-location-section.tsx` | `disabledDate` on both DatePickers |
| `src/pages/events/onboarding/index.tsx` | Add Event title/description header; Save label tweak |
| `src/pages/events/dashboard/index.tsx` | Breadcrumb uses `EVENT_ADD_PAGE_TITLE` |
| `src/components/forms/StateCityFormFields.tsx` | `allowClear` off when disabled; conditional `disabled` spread |
| `src/pages/import-tool/dashboard/components/validate-import-step.tsx` | Empty-CSV error alert; fix false “all rows passed” |
| `src/pages/import-tool/dashboard/constants/import-api.constants.ts` | `IMPORT_MIN_CSV_ROWS` |
| `src/pages/import-tool/dashboard/constants/validate-step.constants.ts` | Empty CSV copy |
| `src/pages/import-tool/dashboard/utils/validation-table.utils.ts` | `hasMinimumImportRows` |
| `src/pages/videos/details/components/DistributeModal.tsx` | Unrelated copy tweak |
| `src/pages/videos/details/components/distribute/DistributeModal.tsx` | Unrelated copy tweak |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `EVENT_MAX_PAST_YEARS` + `eventDatePastRangeMessage` match backend constant/message | ✅ |
| Range logic centralized in `event-schedule-validation.utils.ts`; wired to rules + `disabledDate` + submit guard | ✅ Fixed (#1) |
| Empty CSV: `hasMinimumImportRows` fixes false “all rows passed” on header-only files | ✅ |
| `StateCityFormFields` disabled/clear fix is localized and correct | ✅ |
| City still uses `createEventRequiredTextRules` (`/[a-zA-Z0-9]/`) vs backend letter-only | ⚠️ Finding #1 |
| `DistributeModal` copy changes unrelated to SKSH-349 | ✅ Accepted (minor scope bleed) |
| FE date cutoff uses local `dayjs()`; backend uses UTC — pre-existing event-date stack pattern | ✅ Accepted |

### Positive notes

- Follow-up commit `bef1eaf5` closes the prior High gap: `disabledDate`, form rules, and `getEventScheduleValidationError` all enforce the 25-year floor.
- Empty-CSV handling closes a real bug on header-only uploads.
- Bulk update paths get `pastRangeRule` on start/end rules when dates are provided.

---

## GitHub comments

**`src/pages/events/onboarding/components/schedule-location-section.tsx` (line 30)**  
`cityRules` still uses `createEventRequiredTextRules` (`/[a-zA-Z0-9]/`). Backend `eventCityFieldSchema` requires at least one letter (`/[a-zA-Z]/`) and rejects digit-only values like `"43"`. Please add `createEventCityRules()` with the backend-aligned message and use it here.

---

## Findings

---
City validation pattern mismatches backend letter-only rule

Risk Level: MEDIUM  
File Path: skillshow-admin-ui/src/pages/events/onboarding/components/schedule-location-section.tsx  
Lines: 30

Description:
**DRY / Contract.** `cityRules` uses `createEventRequiredTextRules`, which validates against `EVENT_MEANINGFUL_TEXT_PATTERN` (`/[a-zA-Z0-9]/`). Backend `eventCityFieldSchema` requires `/[a-zA-Z]/` and rejects digit-only values (e.g. `"43"`). The city control is a `Select`, so manual invalid entry is unlikely in normal UI, but the validator and error message still diverge from the API contract and import validation.

Impact:
- Digit-only city values pass client validation but fail on API submit or import.
- Error messages differ: frontend `"Include at least one letter or number."` vs backend `"city must contain at least one letter"`.

Recommendation:
Add `EVENT_CITY_LETTER_PATTERN = /[a-zA-Z]/` and `createEventCityRules()` in `event-field-validation.utils.ts` with message `city must contain at least one letter`. Use it in `schedule-location-section.tsx` instead of `createEventRequiredTextRules("City is required")`.

**GitHub comment (File: `skillshow-admin-ui/src/pages/events/onboarding/components/schedule-location-section.tsx`, Lines: `30`):** City rules still use the generic meaningful-text pattern (allows digits-only). Please align with backend letter-only city validation.

---

## Resolved since initial review

---
Event form 25-year date guard (cross-stack contract)

Risk Level: HIGH  
File Path: skillshow-admin-ui/src/pages/events/utils/event-schedule-validation.utils.ts  
Lines: 17-122

Status: ✅ Fixed (commit `bef1eaf5`)

Evidence:
- `EVENT_MAX_PAST_YEARS` and `eventDatePastRangeMessage` added in `constants.ts`.
- `disabledEventDateBeforeAllowedRange` wired on both DatePickers in `schedule-location-section.tsx`.
- `createEventStartDateRules` / `createEventEndDateRules` include `createEventDatePastRangeRule`.
- `getEventScheduleValidationError` enforces range on submit (create/edit + bulk update).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | City validation pattern mismatches backend letter-only rule | MEDIUM | Accepted | skillshow-admin-ui/src/pages/events/onboarding/components/schedule-location-section.tsx | 30 |

**Merge readiness:** ✅ No Critical or High blockers. Medium city-rule alignment finding marked **Accepted** — low practical risk (city is a `Select`); safe to merge as-is.
