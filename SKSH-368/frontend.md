# PR review (SKSH-368) — skillshow-admin-ui

PR: `https://github.com/SkillshowFx/skillshow-admin-ui/pull/322`  
Base: `main`  
Head: `SKSH-368` @ `cb647cf`  
**Re-review:** 2026-07-01 @ `cb647cf` (unchanged)

Prompt: `pr-review/prompts/frontend-system-prompt.md`

**Aligned with:** [backend.md](./backend.md)

## GitHub comments

### `src/pages/management/app-users/onboarding/utils.ts`
- **MEDIUM** — Parent API errors mapped via loose `/parent/i` regex (line ~286)

## Findings

---
Parent API errors mapped via loose `/parent/i` regex

Risk Level: MEDIUM  
**Status:** Open  
File Path: src/pages/management/app-users/onboarding/utils.ts  
Lines: 286-293

Description:
**KISS / UX.** `mapAppUserCreateErrorToFormField` routes any API error containing "parent" (case-insensitive) to the `parentEmail` field.

**Re-review:** No diff changes since initial review @ `cb647cf`; `/parent/i` mapping still present.

Recommendation:
Map known backend messages from `resolveActiveParentUserByEmail` (missing / wrong_role / inactive) or stable error codes instead of a broad regex.

---

### Cross-stack note

Backend `034a93a2` fixes minor resend/patch/Joi gaps. App-users list **Resend mail** action still shown for all users — now routes correctly server-side for placeholder minors, but UI could relabel for clarity (optional).

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Parent API errors mapped via loose `/parent/i` regex | MEDIUM | Open | `src/pages/management/app-users/onboarding/utils.ts` | 286-293 |

**Merge readiness:** No open Critical/High blockers. One optional MEDIUM DRY/UX cleanup remains.
