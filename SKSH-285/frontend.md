# PR review — SKSH-285 (skillshow-admin-ui)

| Field | Value |
|-------|-------|
| PR | [#320](https://github.com/SkillshowFx/skillshow-admin-ui/pull/320) |
| Branch | `sksh-285` → `main` |
| Scope | `AppDatePicker` / `AppDateRangePicker` wrappers; keyboard-enabled date entry across admin UI |
| Prompt | `pr-review/prompts/frontend-system-prompt.md` |

## GitHub comments

### `src/pages/auth/register-form.tsx` (line 447)

**CRITICAL** — Registration DOB payload format regressed from ISO to MM/DD/YYYY

## Findings

---
Registration DOB payload format regressed from ISO to MM/DD/YYYY

Risk Level: CRITICAL
File Path: src/pages/auth/register-form.tsx
Lines: 447

Description:
The PR changes the registration form `onChange` handler from `date.format("YYYY-MM-DD")` to `date.format(DEFAULT_DATE_FORMAT)` (`MM/DD/YYYY`). The register mutation still sends `dob: fields.dob` unchanged. Backend registration validation requires `Joi.string().isoDate()` (`auth.validation.ts`), which only accepts ISO `YYYY-MM-DD`. **Contract** violation — keyboard-entry standardization must not change the wire format for this endpoint.

Impact:
- New user registration with a DOB will fail API validation (`400` — "Date of birth must be in valid ISO format (YYYY-MM-DD)").
- Users can type or pick a valid date in the UI but cannot complete signup.

Recommendation:
Keep `AppDatePicker` for keyboard entry and display, but preserve ISO storage for the register payload:

```tsx
onChange={(date) => {
  setFields((prev) => ({
    ...prev,
    dob: date ? date.format("YYYY-MM-DD") : "",
  }));
}}
```

And keep parsing the controlled value with the ISO format:

```tsx
value={fields.dob ? parseDateWithFormat(fields.dob, "YYYY-MM-DD") : null}
```

Only the display/typing format should use `MM/DD/YYYY`; the API contract stays `YYYY-MM-DD`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Registration DOB payload format regressed from ISO to MM/DD/YYYY | CRITICAL | Open | src/pages/auth/register-form.tsx | 447 |

**Merge readiness:** Blocked — registration will fail backend `isoDate` validation until the DOB wire format is restored to `YYYY-MM-DD`.
