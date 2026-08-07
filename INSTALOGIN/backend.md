# Backend PR Review — skillshow (`INSTALOGIN`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/260  
**Branch:** `feat/insta-login` → main  
**Head:** `70cf586b87e2d892823630f2b1e498035a13b6ad`  
**Scope:** Migrate Instagram OAuth from Facebook Login to Business Login for Instagram (Instagram Login); long-lived token exchange, refresh, profile enrichment, legacy reconnect signaling  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired:** `pr-review/INSTALOGIN/frontend.md` (#372), `pr-review/INSTALOGIN/orchestrator.md` (#30)  
**Updated:** 2026-08-07 — re-verify on latest head (findings #1–#2 fixed)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

---
Debug log emits OAuth client secret

Risk Level: HIGH
File Path: src/utils/vendor/handlers/index.ts
Lines: 103

Description:
**Security.** Debug `console.log("authParams", authParams)` removed from `getAuthorizationUrl`.

Impact:
- Resolved — client secret no longer logged on connect attempts.

Recommendation:
N/A — fixed on head `70cf586b`.
---

---
Silent fallback to short-lived token when long-lived exchange fails

Risk Level: HIGH
File Path: src/utils/vendor/handlers/social/instagram.handler.ts
Lines: 161-188

Description:
Long-lived `ig_exchange_token` is now required; missing token/secret or exchange failure throws instead of returning short-lived-only credentials.

Impact:
- Resolved — connections are not persisted as linked when long-lived exchange fails.

Recommendation:
N/A — fixed on head `70cf586b`.
---

**Positive notes:** `needsReconnect` signaling for legacy Facebook-Login Instagram tokens. OAuth state via Redis. Instagram Login scopes and token URLs align with orchestrator #30.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Debug log emits OAuth client secret | HIGH | ✅ Fixed | src/utils/vendor/handlers/index.ts | 103 |
| 2 | Silent fallback to short-lived token when long-lived exchange fails | HIGH | ✅ Fixed | src/utils/vendor/handlers/social/instagram.handler.ts | 161-188 |

**Merge readiness:** **Merge-ready** — ship with frontend #372 once orchestrator #30 clears `.env.dev` blocker.
