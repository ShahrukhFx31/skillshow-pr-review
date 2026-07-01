# PR review (SKSH-368) — skillshow

PR: `https://github.com/SkillshowFx/skillshow/pull/223`  
Base: `main`  
Head: `SKSH-368` @ `5df2d91`

Prompt: `pr-review/prompts/backend-system-prompt.md`

**Aligned with:** [frontend.md](./frontend.md)

## GitHub comments

### `src/validation/app-user.validation.ts`
- **HIGH** — Create schema does not conditionally require `parentEmail` / athlete `email` (line 32)

### Summary-only (not in PR diff hunks)
- **HIGH** — `resendWelcomeEmail` targets placeholder minor email (`src/services/app-user.service.ts` ~534-539)
- **HIGH** — `patchAppUser` allows email overwrite on minor athletes (`src/services/app-user.service.ts` ~549-560)

## Findings

---
`resendWelcomeEmail` sends to minor placeholder address

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 534-539

Description:
**Contract / operational correctness.** Minor athletes created via `createMinorAthleteAppUser` store a system placeholder on `user.email` (`*@minor.skillshow.local`) and deliver credentials to the parent via `athleteService.create`. `resendWelcomeEmail` still calls `sendWelcomeEmail(user, …)`, which uses `user.email` only.

Impact:
- Admin "Resend welcome email" on a minor athlete sends to an undeliverable placeholder (or fails silently depending on mail config).
- Parent never receives replacement credentials; support must intervene manually.

Recommendation:
Detect minor athlete accounts (placeholder suffix and/or `resolveAthleteAccountDob` + `isAthleteMinorByDob`). For minors, resend via the parent-linked credential path (resolve parent from athlete `parents` / linked users and call the same email helper used in `AthleteService.create`), or return `400` with a clear message if resend is not supported. Add a service test for the minor branch.

---

PR comment (inline):
Minor athletes use a placeholder `user.email`, but `resendWelcomeEmail` still calls `sendWelcomeEmail` with that address. Admin resend will not reach the parent who actually receives credentials on create. Branch on minor/placeholder accounts and resend to the linked parent (or reject with a clear 400).

---
`patchAppUser` can overwrite minor placeholder email via API

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 549-560

Description:
**Security / data integrity (admin defense in depth).** The new minor-athlete create flow relies on placeholder login emails. `patchAppUser` still applies `body.email` unconditionally when present. The admin UI omits email for minors, but the admin API accepts direct PATCH bodies.

Impact:
- A mistaken or scripted PATCH can replace the placeholder email and break the minor login model or parent/child linkage expectations.
- No server-side guard when an athlete is still under 13 / uses a placeholder email.

Recommendation:
Before applying `body.email`, resolve whether the target user is a minor athlete (placeholder suffix or DOB). Reject email changes while still minor unless transitioning to 13+ with a validated real email. Mirror the frontend rule: minors should not have `email` / `parentEmail` / `phone` patched through this endpoint.

---

PR comment (inline):
`patchAppUser` still writes `body.email` for any user. Minor athletes depend on placeholder emails; the UI strips email on patch but the API does not. Reject email (and parent/phone) patches for minor accounts unless you are explicitly handling a 13+ transition with validation.

---
Create Joi schema allows athlete bodies with neither `email` nor `parentEmail`

Risk Level: HIGH  
File Path: src/validation/app-user.validation.ts  
Lines: 32-38

Description:
**Contract.** `createAppUserSchema` makes `email` optional and adds optional `parentEmail`, but there is no Joi `.when()` tying athlete DOB to required `parentEmail` (under 13) or `email` (13+). Service-layer checks catch this today, but validation middleware should enforce the same contract so invalid payloads fail consistently at the route boundary.

Impact:
- Invalid create bodies pass Joi and fail later with ad-hoc 400 messages.
- Future service changes could accidentally skip a branch and accept malformed creates.

Recommendation:
Add conditional Joi rules on `createAppUserSchema`, e.g. when `role === athlete` and parsed DOB is under `ATHLETE_MIN_EMAIL_AGE_YEARS`, require `parentEmail`; when 13+, require `email`. Keep service checks as defense in depth.

---

PR comment (inline):
`email` and `parentEmail` are both optional in Joi, so athlete creates with neither field pass validation middleware. Service catches this, but the route contract should use `.when()` on role/DOB to require `parentEmail` for under-13 athletes and `email` for 13+. Align Joi with `createAppUser` service rules.

### Positive notes

- Reuses `athleteService.create` for admin minor import — consistent with parent-initiated flow, parent RBAC validation via `resolveActiveParentUserByEmail`.
- `resolveAthleteAccountDob` fixes stale minor/major detection when `profile.dob` and legacy `athlete.dob` diverge (profile, athlete, share-permission paths updated).
- Athlete search supplements user text match with profile-name lookup; regex escaped via `containsInsensitive`.
- Immediate-connection notifications limited to in-app for parent-created minors — avoids duplicate welcome email.
- Solid test coverage for minor create, parent validation, search merge, and relation notify.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `resendWelcomeEmail` sends to minor placeholder address | HIGH | Open | `src/services/app-user.service.ts` | 534-539 |
| 2 | `patchAppUser` can overwrite minor placeholder email via API | HIGH | Open | `src/services/app-user.service.ts` | 549-560 |
| 3 | Create Joi schema missing conditional athlete email/parentEmail rules | HIGH | Open | `src/validation/app-user.validation.ts` | 32-38 |

**Merge readiness:** Open Critical/High blockers — minor-athlete admin lifecycle gaps on resend/patch/validation.
