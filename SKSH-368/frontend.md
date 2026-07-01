# PR review (SKSH-368) — skillshow-admin-ui

PR: `https://github.com/SkillshowFx/skillshow-admin-ui/pull/322`  
Base: `main`  
Head: `SKSH-368` @ `cb647cf`

Prompt: `pr-review/prompts/frontend-system-prompt.md`

**Aligned with:** [backend.md](./backend.md)

## GitHub comments

### `src/pages/management/app-users/onboarding/utils.ts`
- **MEDIUM** — Parent API errors mapped via loose `/parent/i` regex (line ~286)

## Findings

---
Parent API errors mapped via loose `/parent/i` regex

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/onboarding/utils.ts  
Lines: 286-293

Description:
**KISS / UX.** `mapAppUserCreateErrorToFormField` routes any API error containing "parent" (case-insensitive) to the `parentEmail` field. Backend messages are specific today, but unrelated 400/500 text that mentions "parent" could highlight the wrong field.

Impact:
- Misleading inline validation on minor-athlete create when an unexpected error occurs.
- Harder debugging if error copy changes.

Recommendation:
Map known backend error codes or stable message prefixes from `resolveActiveParentUserByEmail` (missing / wrong_role / inactive) instead of a broad regex. Fall back to a form-level Alert when the message is unrecognized.

---

PR comment (inline):
`mapAppUserCreateErrorToFormField` treats any error message matching `/parent/i` as a `parentEmail` field error. That is brittle if copy changes or another failure mentions "parent". Match explicit backend cases (missing / wrong role / inactive) or error codes.

### Cross-stack note (backend owns fix; optional UI follow-up)

The app-users list still exposes **Resend mail** for all users (`app-users-columns.tsx`). Minor athletes use placeholder emails server-side — see backend finding #1. Consider hiding or relabeling that action for minor placeholder accounts once backend resend behavior is fixed.

### Positive notes

- Minor-athlete add form UX is thorough: DOB-driven field swap, contact draft preservation, info alerts, and `parentEmail` payload wiring align with backend create contract.
- `MINOR_ATHLETE_PLACEHOLDER_EMAIL_SUFFIX` mirrors backend constant.
- `buildAppUserPatchPayload` strips `email` / `parentEmail` / `phone` for minors — matches read-only intent.
- Athlete onboarding search `staleTime: 0` helps reflect expanded name search + fresh connection status after backend SKSH-368 changes.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Parent API errors mapped via loose `/parent/i` regex | MEDIUM | Open | `src/pages/management/app-users/onboarding/utils.ts` | 286-293 |

**Merge readiness:** No open Critical/High/Medium blockers on changed frontend files. Address MEDIUM error-mapping cleanup when convenient; coordinate with backend on resend-welcome for minors.
