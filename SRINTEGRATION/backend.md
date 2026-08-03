# Backend PR Review — skillshow (`SRINTEGRATION`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/259  
**Branch:** `feat/srintegration` → main  
**Head:** `4e55a919b49572e94e4d1bd57b4f9756c4afbabd`  
**Scope:** SportsRecruits vendor — account-linking start URL, webhook handler, API vendor constants, env config  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired:** `pr-review/SRINTEGRATION/frontend.md` (#371), `pr-review/SRINTEGRATION/orchestrator.md` (#29)

## Developer replies (review threads)

| Finding | Developer (`dhanraj-fx31labs`) | Re-verification |
|---------|-------------------------------|-----------------|
| Webhook logs full headers/body | "this is for testing purpose" | Still present at head — remove before merge |
| Webhook signature / IDOR | _(no reply; thread resolved)_ | Fixed — `verifySportsRecruitsWebhookSignature` + `consumePendingLink` |

## GitHub comments

### `src/controllers/vendor.controller.ts`

- **L267** — Webhook still logs full headers/body (PII / secrets)

## Findings

---
SportsRecruits webhook has no signature verification

Risk Level: CRITICAL
File Path: src/controllers/vendor.controller.ts
Lines: 246-286

Description:
**Security.** `POST /v1/vendors/webhooks/sportsrecruits` is excluded from JWT (`app.ts`) but the handler never validates inbound authenticity. `buildSportsRecruitsSignature` in `sportsrecruits/auth.ts` is only used for outbound link URLs, not webhooks.

Impact:
- Any caller can forge `account.linked` payloads and activate vendor connections for arbitrary users.

Recommendation:
Verify each webhook per SportsRecruits docs (HMAC/`authorization` over canonical JSON using server-side partner id + token). Reject with 401/403 before calling `handleSportsRecruitsAccountLinked`.

**Re-verify (f480d3b):** ✅ Implemented — `verifySportsRecruitsWebhookSignature` over `rawBody` / canonical JSON; 401 on mismatch. Tests in `tests/utils/sportsrecruits-auth.test.ts`.
---

---
Webhook trusts `partner_user_id` without correlation to pending link (IDOR)

Risk Level: CRITICAL
File Path: src/services/vendor.service.ts
Lines: 317-332

Description:
**Security / IDOR.** `handleSportsRecruitsAccountLinked` sets `userId: partnerUserId` from the request body and calls `createOrUpdate` with `linkingStatus: "linked"`. `connectSportsRecruitsStart` correctly uses the authenticated user's id, but the webhook does not prove SportsRecruits sent that id.

Impact:
- Attacker can link SportsRecruits (or fake metadata) to any Skillshow user id and trigger vendor-connected notifications.

Recommendation:
After signature verification, require a server-issued pending link record from `connectSportsRecruitsStart` (expiry/nonce) and only activate when `partner_user_id` matches.

**Re-verify (f480d3b):** ✅ Fixed — `SportsRecruitsPendingLinkService` (Redis + memory fallback); `createPendingLink` on start, `consumePendingLink` on webhook; throws `PENDING_LINK_REQUIRED` when missing.
---

---
Webhook logs full headers and body

Risk Level: HIGH
File Path: src/controllers/vendor.controller.ts
Lines: 262-268

Description:
**Security.** `sportsRecruitsWebhook` logs `query`, `headers`, and `body` at info level, which can include authorization material, cookies, emails, and profile URLs.

Impact:
- Credential and PII leakage in production logs.

Recommendation:
Log allowlisted fields only (event type, hashed ids, request id). Remove full header/body logging.

**Developer reply:** "this is for testing purpose" — acceptable for local debugging only; must not ship to production/staging logs.
---

---
Authenticated users can self-connect via POST /connect/api with partner credentials

Risk Level: HIGH
File Path: src/validation/vendor.validation.ts
Lines: 30-50

Description:
**Security / Contract.** `connectApiSchema` allows `vendorName: sportsrecruits` with client-supplied `credentialsRaw.partnerId` and `token`. Users can persist partner-level API secrets in per-user connection records, bypassing the intended redirect + webhook flow.

Impact:
- Global partner secrets may be stored per user; undermines webhook-only linking model.

Recommendation:
Disallow `sportsrecruits` on `POST /connect/api` (dedicated GET start + webhook only). Keep `SR_*` env server-side only.

**Re-verify (f480d3b):** ✅ Fixed — Joi forbids partner secrets; `connectApi` throws `ACCOUNT_LINKING_REQUIRED` for SR.
---

---
No pending-link state between start and webhook

Risk Level: HIGH
File Path: src/services/vendor.service.ts
Lines: 281-310

Description:
**Contract.** `connectSportsRecruitsStart` returns a redirect URL but does not persist a pending connection. The webhook cannot correlate events to a user-initiated, time-bound session.

Impact:
- Harder to detect forged or replayed webhook events even after signature verification is added.

Recommendation:
Create/update connection with `linkingStatus: "pending"` and metadata (startedAt, nonce); webhook transitions to `linked` only for matching pending rows.

**Re-verify (f480d3b):** ✅ Fixed — pending-link service with 15m TTL.
---

---
Webhook JWT bypass uses path.endsWith suffix matching

Risk Level: HIGH
File Path: src/app.ts
Lines: 133-136

Description:
**Security.** `conditionalAuth` skips JWT when `path.endsWith("/v1/vendors/webhooks/sportsrecruits")`. Suffix matching can unintentionally expose unrelated routes whose path ends with that segment (see `SECURITY-AUDIT-PRE-RELEASE.md` — prefer exact path equality).

Impact:
- Broader unauthenticated surface than intended if future routes share the suffix.

Recommendation:
Use exact match only: `path === "/v1/vendors/webhooks/sportsrecruits"` (and normalized path without query string if needed).

**Re-verify (4e55a919):** ✅ Fixed — `endsWith` removed; exact path match only in `conditionalAuth`.
---

---
connectApiStart returns wrong success message constant

Risk Level: MEDIUM
File Path: src/controllers/vendor.controller.ts
Lines: 229

Description:
Returns `VENDOR_SUCCESS_MESSAGES.LIST` instead of a connect/redirect-specific message.

Recommendation:
Add or reuse an appropriate success constant for API connect start.

**Re-verify (f480d3b):** ✅ Fixed — uses `VENDOR_SUCCESS_MESSAGES.CONNECT_STARTED`.
---

**Positive notes:** Outbound HMAC link URL builder aligns with SR docs; raw body capture for signature verification; webhook signature tests added; `connectSportsRecruitsStart` binds `partner_user_id` to authenticated user.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | SportsRecruits webhook has no signature verification | CRITICAL | ✅ Fixed | src/controllers/vendor.controller.ts | 246-286 |
| 2 | Webhook trusts partner_user_id without pending-link correlation (IDOR) | CRITICAL | ✅ Fixed | src/services/vendor.service.ts | 317-332 |
| 3 | Webhook logs full headers and body | HIGH | Open | src/controllers/vendor.controller.ts | 262-268 |
| 4 | POST /connect/api allows client-supplied partner credentials | HIGH | ✅ Fixed | src/validation/vendor.validation.ts | 30-50 |
| 5 | No pending-link state between start and webhook | HIGH | ✅ Fixed | src/services/vendor.service.ts | 281-310 |
| 6 | connectApiStart returns wrong success message constant | MEDIUM | ✅ Fixed | src/controllers/vendor.controller.ts | 229 |
| 7 | Webhook JWT bypass uses path.endsWith suffix matching | HIGH | ✅ Fixed | src/app.ts | 133 |

**Merge readiness:** **Not merge-ready** — remove verbose webhook logging (developer: "for testing purpose") before merge.
