# Backend PR Review — skillshow (`INSTALOGIN`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/260  
**Branch:** `feat/insta-login` → main  
**Head:** `fbfc00b685a2bb37625dd3c55a839910e4a25a74`  
**Scope:** Migrate Instagram OAuth from Facebook Login to Business Login for Instagram (Instagram Login); long-lived token exchange, refresh, profile enrichment, legacy reconnect signaling  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired:** `pr-review/INSTALOGIN/frontend.md` (#372), `pr-review/INSTALOGIN/orchestrator.md` (#30)

## GitHub comments

### `src/utils/vendor/handlers/index.ts`

- **L103** — Debug log emits OAuth client secret

### `src/utils/vendor/handlers/social/instagram.handler.ts`

- **L181** — Silent fallback to short-lived token when long-lived exchange fails

## Findings

---
Debug log emits OAuth client secret

Risk Level: HIGH
File Path: src/utils/vendor/handlers/index.ts
Lines: 103

Description:
**Security.** `getAuthorizationUrl` adds `console.log("authParams", authParams)` where `authParams` includes `clientSecret`. This will write app secrets to stdout/log aggregators on every Instagram connect attempt.

Impact:
- Instagram App Secret exposure in application logs (credential leak).
- Violates secrets-in-logs policy from `SECURITY-AUDIT-PRE-RELEASE.md`.

Recommendation:
Remove the `console.log` entirely. If debug is needed, log only non-sensitive fields (`vendorName`, `redirectUri`, scope list) via `logger.debug` behind a development flag — never log `clientSecret` or `codeVerifier`.
---

---
Silent fallback to short-lived token when long-lived exchange fails

Risk Level: HIGH
File Path: src/utils/vendor/handlers/social/instagram.handler.ts
Lines: 161-183

Description:
After the authorization-code exchange, `exchangeCodeForTokens` wraps the long-lived `ig_exchange_token` call in a try/catch that **returns the short-lived token on any failure**. Short-lived Instagram Login tokens expire in ~1 hour and cannot be refreshed the same way as long-lived tokens. The user appears connected (`linkingStatus: linked`) but publish/insights will fail soon after.

Impact:
- False-positive “connected” state when Meta exchange fails (misconfigured secret, network timeout, App Review scope gaps).
- Distribution and insights jobs fail intermittently with opaque Graph API auth errors.
- Token refresh path assumes long-lived semantics (`ig_refresh_token`).

Recommendation:
Treat long-lived exchange as required for Instagram Login. On exchange failure, throw (or return a structured OAuth error to the callback) so the connection is not persisted as linked. Surface a user-facing “Could not complete Instagram connection — try again” message. Keep the existing `IG_EXCHANGE_TIMEOUT_MS` and log the failure via `logger.error` without storing short-lived-only credentials.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Debug log emits OAuth client secret | HIGH | Open | src/utils/vendor/handlers/index.ts | 103 |
| 2 | Silent fallback to short-lived token when long-lived exchange fails | HIGH | Open | src/utils/vendor/handlers/social/instagram.handler.ts | 161-183 |

**Merge readiness:** Not merge-ready — remove debug logging and fail closed on long-lived token exchange before shipping.
