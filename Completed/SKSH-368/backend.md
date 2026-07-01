# PR review (SKSH-368) — skillshow

PR: `https://github.com/SkillshowFx/skillshow/pull/223`  
Base: `main`  
Head: `SKSH-368` @ `034a93a2`  
**Re-review:** 2026-07-01 @ `034a93a2` (unchanged)

Prompt: `pr-review/prompts/backend-system-prompt.md`

**Aligned with:** [frontend.md](./frontend.md)

## GitHub comments

No new inline comments — prior High findings resolved in `034a93a2`.

## Findings

---
`resendWelcomeEmail` sends to minor placeholder address

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/services/app-user.service.ts  
Lines: 544-556

Description:
Minor athletes use placeholder `user.email`; `resendWelcomeEmail` previously emailed that address.

**Re-review evidence:** `034a93a2` branches on `isMinorPlaceholderEmail(user.email)` for athletes and calls `athleteService.resendMinorAthleteParentCredentials`.

---

---
`patchAppUser` can overwrite minor placeholder email via API

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/services/app-user.service.ts  
Lines: 569-606

Description:
`patchAppUser` applied `body.email` unconditionally for minor athletes.

**Re-review evidence:** Placeholder minors reject email patch unless DOB transitions athlete to 13+ with validated real email.

---

---
Create Joi schema missing conditional athlete email/parentEmail rules

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/validation/app-user.validation.ts  
Lines: 89-131

Description:
`createAppUserSchema` allowed athlete bodies with neither `email` nor `parentEmail` at Joi.

**Re-review evidence:** `.custom()` validator requires athlete DOB, `parentEmail` when under 13, and `email` when 13+.

---

### Optional follow-up (not reported as findings)

- `patchAppUser` still accepts `phone` for placeholder minors via API (UI omits it); low risk.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `resendWelcomeEmail` sends to minor placeholder address | HIGH | ✅ Fixed | `src/services/app-user.service.ts` | 544-556 |
| 2 | `patchAppUser` can overwrite minor placeholder email via API | HIGH | ✅ Fixed | `src/services/app-user.service.ts` | 569-606 |
| 3 | Create Joi schema missing conditional athlete email/parentEmail rules | HIGH | ✅ Fixed | `src/validation/app-user.validation.ts` | 89-131 |

**Merge readiness:** No open Critical/High blockers.
