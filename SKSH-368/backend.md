# PR review (SKSH-368) — skillshow

PR: `https://github.com/SkillshowFx/skillshow/pull/223`  
Base: `main`  
Head: `SKSH-368` @ `034a93a2`  
**Re-review:** 2026-07-01 @ `034a93a2` (was `5df2d91`)

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
Lines: 544-556 (was 534-539)

Description:
Minor athletes use placeholder `user.email`; `resendWelcomeEmail` previously emailed that address.

**Re-review evidence:** `034a93a2` branches on `isMinorPlaceholderEmail(user.email)` for athletes and calls `athleteService.resendMinorAthleteParentCredentials` (resolves legal parent, sends credential email). Test: `resends minor athlete credentials to parent instead of placeholder email`.

---

---
`patchAppUser` can overwrite minor placeholder email via API

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/services/app-user.service.ts  
Lines: 569-606 (was 549-560)

Description:
`patchAppUser` applied `body.email` unconditionally, allowing placeholder email overwrite for minor athletes.

**Re-review evidence:** Placeholder minor athletes reject email patch unless DOB in the same request transitions the athlete to 13+ with a validated real email. Test: `rejects email patch for under-13 placeholder athlete`.

---

---
Create Joi schema missing conditional athlete email/parentEmail rules

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/validation/app-user.validation.ts  
Lines: 89-131 (was 32-38)

Description:
`createAppUserSchema` allowed athlete bodies with neither `email` nor `parentEmail` at the Joi layer.

**Re-review evidence:** `.custom()` validator requires athlete DOB, `parentEmail` when under 13, and `email` when 13+; non-athlete roles still require `email`. New tests in `app-user.validation.test.ts`.

---

### Positive notes (re-review)

- `resendMinorAthleteParentCredentials` + `resolveLegalParentUserIdForAthleteUser` centralize minor resend logic for reuse from app-user admin.
- `isMinorPlaceholderEmail` extracted for consistent placeholder detection.
- Joi custom messages align with service-layer error copy.

### Optional follow-up (not reported as findings)

- `patchAppUser` still accepts `phone` for placeholder minors via API (UI omits it); low risk but could mirror email guard if desired.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `resendWelcomeEmail` sends to minor placeholder address | HIGH | ✅ Fixed | `src/services/app-user.service.ts` | 544-556 |
| 2 | `patchAppUser` can overwrite minor placeholder email via API | HIGH | ✅ Fixed | `src/services/app-user.service.ts` | 569-606 |
| 3 | Create Joi schema missing conditional athlete email/parentEmail rules | HIGH | ✅ Fixed | `src/validation/app-user.validation.ts` | 89-131 |

**Merge readiness:** No open Critical/High blockers on backend. Frontend MEDIUM error-mapping still open (see `frontend.md`).
