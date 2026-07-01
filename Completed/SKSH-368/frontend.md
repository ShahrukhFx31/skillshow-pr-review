# PR review (SKSH-368) — skillshow-admin-ui

PR: `https://github.com/SkillshowFx/skillshow-admin-ui/pull/322`  
Base: `main`  
Head: `SKSH-368` @ `bdacdef8`  
**Re-review:** 2026-07-01 @ `bdacdef8` (was `cb647cf`)

Prompt: `pr-review/prompts/frontend-system-prompt.md`

**Aligned with:** [backend.md](./backend.md)

## GitHub comments

No new inline comments — prior MEDIUM finding resolved in `bdacdef8`.

## Findings

---
Parent API errors mapped via loose `/parent/i` regex

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/management/app-users/onboarding/utils.ts  
Lines: 286-318 (was 286-293)

Description:
**KISS / UX.** `mapAppUserCreateErrorToFormField` routed any API error containing "parent" to the `parentEmail` field.

**Re-review evidence:** `bdacdef8` replaces `/parent/i` with `PARENT_EMAIL_FIELD_API_ERROR_PATTERNS` — explicit regexes aligned to backend `resolveActiveParentUserByEmail` and Joi messages (`isParentEmailFieldApiError`). Comment documents cross-repo alignment requirement.

---

### Positive notes (re-review)

- **Resend label:** `app-users-columns.tsx` shows "Resend credentials to parent" for placeholder minor athletes (cross-stack UX note addressed).
- Minor-athlete form flow unchanged and still aligned with backend create contract.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Parent API errors mapped via loose `/parent/i` regex | MEDIUM | ✅ Fixed | `src/pages/management/app-users/onboarding/utils.ts` | 286-318 |

**Merge readiness:** No open Critical/High/Medium blockers.
