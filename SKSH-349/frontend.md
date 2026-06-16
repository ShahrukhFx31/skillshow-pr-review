# Frontend PR Review — skillshow-admin-ui (`SKSH-349-1`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-349-1`  
**Base:** `main...HEAD` @ `1da9ca66`  
**Initial review:** 2026-06-16  
**Scope:** Event add-page header copy, import-tool empty-CSV guard, `StateCityFormFields` disabled/clear behavior; cross-check backend date/city validation (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 2 (0 Critical, 1 High, 1 Medium) — **2 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/events/onboarding/index.tsx` | Add Event title/description header; Save label tweak |
| `src/pages/events/constants.ts` | `EVENT_ADD_PAGE_TITLE` / `EVENT_ADD_PAGE_DESCRIPTION` |
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
| Empty CSV: `hasMinimumImportRows` + `IMPORT_MIN_CSV_ROWS` — fixes `invalidCount === 0` false success on header-only files | ✅ |
| Import footer already blocks `validCount === 0`; new alert improves UX | ✅ |
| `StateCityFormFields` disabled/clear fix is localized and correct | ✅ |
| Event date 25-year rule not mirrored in `event-schedule-validation.utils.ts` or DatePicker | ⚠️ Finding #1 |
| City uses `createEventRequiredTextRules` (`/[a-zA-Z0-9]/`) vs backend letter-only | ⚠️ Finding #2 |
| `DistributeModal` copy changes unrelated to SKSH-349 | ✅ Accepted (minor scope bleed) |

### Positive notes

- Empty-CSV handling closes a real bug: before this PR, `allRowsValid = invalidCount === 0` showed “All rows passed” for `totalRows === 0`.
- Add Event page header (`CardTitle` / `CardDescription`) matches dashboard banner pattern.
- Partial-import hint now only shows when both valid and invalid rows exist (`validCount > 0 && invalidCount > 0`).

---

## GitHub comments

_Post on a file **in this PR diff** (findings target code that was **not** changed — see “PR diff” column below)._

**`src/pages/events/constants.ts` (near `EVENT_ADD_PAGE_TITLE`)**  
Backend `SKSH-349` rejects `startDate`/`endDate` older than 25 years. Follow-up is needed in `schedule-location-section.tsx` + `event-schedule-validation.utils.ts` (not modified in this PR) — please add `EVENT_MAX_PAST_YEARS = 25` here and wire `disabledDate` / rules so the form matches the API before submit.

**`src/pages/events/constants.ts` (same area)**  
Backend city validation is letter-only (`/[a-zA-Z]/`). Follow-up is needed in `schedule-location-section.tsx` (not modified in this PR). Add `createEventCityRules()` aligned to the backend message when you wire the validation parity.

---

## Findings

---
Event form missing 25-year date guard (cross-stack contract)

Risk Level: HIGH  
PR diff: **Not in this PR** (gap / follow-up)  
Recommended fix location: `skillshow-admin-ui/src/pages/events/onboarding/components/schedule-location-section.tsx` (DatePickers ~57–85)  
Related PR touchpoint: `src/pages/events/constants.ts` (in diff — add shared constant here)

Description:
**Contract / Global consistency.** Backend `SKSH-349` adds `eventDateFieldSchema` with `isEventDateWithinImportRange` (25-year floor via `EVENT_IMPORT_MAX_PAST_YEARS`). The frontend PR does **not** modify schedule validation or `schedule-location-section.tsx`. Existing `DatePicker` components still have no `disabledDate`; `createEventStartDateRules` / `createEventEndDateRules` only enforce required + end ≥ start. Users can pick dates such as `02/12/1900` and only see errors after API submit (import validation catches CSV rows; create/edit form does not).

Impact:
- Create/edit submit fails late with backend message instead of inline form feedback.
- `sortBy`/pagination-style cross-stack drift: UI allows values the API rejects (400).
- Bulk update date fields have the same gap when values are entered.

Recommendation:
Add to `src/pages/events/constants.ts`:

```ts
/** Must match backend `EVENT_IMPORT_MAX_PAST_YEARS` in skillshow. */
export const EVENT_MAX_PAST_YEARS = 25;
export const eventDatePastRangeMessage = (field: "startDate" | "endDate") =>
  `${field} must be within the last ${EVENT_MAX_PAST_YEARS} years`;
```

In `event-schedule-validation.utils.ts`, add `isEventDateWithinAllowedRange(dayjs)` and a rule used by start/end rules. On `DatePicker`, set `disabledDate` to disable dates before `dayjs().subtract(EVENT_MAX_PAST_YEARS, "year").startOf("day")` (use same UTC/calendar semantics as `parseEventDateForUi`).

**GitHub comment (File: `skillshow-admin-ui/src/pages/events/constants.ts`, Lines: `53-56`):** Backend now rejects event dates older than 25 years. Follow up by adding DatePicker `disabledDate` rules in `schedule-location-section.tsx` (not modified in this PR) so the form fails fast with the same message.

---
City validation pattern mismatches backend letter-only rule

Risk Level: MEDIUM  
PR diff: **Not in this PR** (gap / follow-up)  
Recommended fix location: `skillshow-admin-ui/src/pages/events/onboarding/components/schedule-location-section.tsx` (`cityRules` ~26)  
Related PR touchpoint: `src/pages/events/constants.ts` (in diff — optional shared pattern/message)

Description:
**DRY / Contract.** `cityRules` uses `createEventRequiredTextRules`, which validates against `EVENT_MEANINGFUL_TEXT_PATTERN` (`/[a-zA-Z0-9]/`). Backend `eventCityFieldSchema` requires `/[a-zA-Z]/` and rejects digit-only values (e.g. `"43"`). This PR does not change `schedule-location-section.tsx`. The city control is a `Select`, so manual entry is limited, but the validator still diverges from the API contract and from CSV/import error text.

Impact:
- Inconsistent validation if city value is set programmatically or via bulk/import flows surfaced in the form.
- Error messages differ: frontend `"Include at least one letter or number."` vs backend `"city must contain at least one letter"`.

Recommendation:
Add `EVENT_CITY_LETTER_PATTERN = /[a-zA-Z]/` and `createEventCityRules()` in `event-field-validation.utils.ts` with the backend-aligned message. Use it in `schedule-location-section.tsx` instead of `createEventRequiredTextRules("City is required")`.

**GitHub comment (File: `skillshow-admin-ui/src/pages/events/constants.ts`, Lines: `53-56`):** City rules still allow digits-only via the generic meaningful-text pattern. Follow up by aligning city validation in `schedule-location-section.tsx` (not modified in this PR) with backend letter-only city validation.

---

## Summary

| # | Title | Risk | Status | PR diff | Fix location |
|---|--------|------|--------|---------|--------------|
| 1 | Event form missing 25-year date guard (cross-stack contract) | HIGH | Open | Not changed | `schedule-location-section.tsx` (~57–85); constant in `events/constants.ts` ✅ in PR |
| 2 | City validation pattern mismatches backend letter-only rule | MEDIUM | Open | Not changed | `schedule-location-section.tsx` (~26) |

**Merge readiness:** Backend validation is in place; import empty-CSV UX is solid. One **High** cross-stack gap on event date range client-side validation — recommend fixing or explicitly **Accepted** before merge if product accepts API-only enforcement.
