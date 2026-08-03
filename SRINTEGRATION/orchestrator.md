# Orchestrator PR Review — skillshow-distribution-orchestrator (`SRINTEGRATION`)

**Repo:** SkillshowFx/skillshow-distribution-orchestrator  
**PR:** https://github.com/SkillshowFx/skillshow-distribution-orchestrator/pull/29  
**Branch:** `feat/srintegration` → main  
**Head:** `a15fc52c947ed6845d126919a1a49ed8e39eacf6`  
**Scope:** SportsRecruits vendor service (HMAC, presigned publish), worker wiring, env/config, test mock refactor  
**Prompt:** `pr-review/prompts/orchestrator-system-prompt.md`  
**Paired:** `pr-review/SRINTEGRATION/backend.md` (#259), `pr-review/SRINTEGRATION/frontend.md` (#371)

## Developer replies (review threads)

| Finding | Developer (`dhanraj-fx31labs`) | Re-verification |
|---------|-------------------------------|-----------------|
| Live secrets in `.env.dev` | "acceptable" | Secrets still in tracked diff at head — must rotate/remove per security policy |

## GitHub comments

### `.env.dev`

- **L48** — Live AWS + SportsRecruits secrets still committed in tracked file

## Findings

---
Live secrets committed in tracked `.env.dev`

Risk Level: CRITICAL
File Path: .env.dev
Lines: 37-39, 73-74

Description:
**Security.** PR updates tracked `.env.dev` with new AWS access key/secret, bucket, and `SR_TOKEN_TEST` / partner id values. `.env` is gitignored but `.env.dev` is not.

Impact:
- Credential exposure in git history; requires rotation before any merge.

Recommendation:
Remove secrets from the diff; rotate all exposed credentials; use placeholders in `.env.dev` or stop tracking it (document via `.env.example` only).

**Developer reply:** "acceptable" — does not mitigate git history exposure or rotation requirement.

**Re-verify (a15fc52c):** Partially fixed — SR (`test_partner_id`/`test_token`) and YouTube (`test_client_*`) now placeholders. **AWS access key + secret in diff still appear to be live credentials** (`AKIA…` / real secret string) — must replace with placeholders and rotate exposed keys.
---

---
Post-publish status check never receives SportsRecruits credentials

Risk Level: HIGH
File Path: src/workers/VendorStatusCheckWorker.ts
Lines: 129-132

Description:
**Global consistency.** After publish, status verification runs for all vendors. `getCredentials()` has no `SPORTSRECRUITS` branch and returns `{}`. `SportsRecruitsVendorService.checkPublishStatus` requires `partnerUserId` → jobs retry until verification_timeout despite successful POST.

Impact:
- Distribution jobs appear stuck/failed in UI after successful SR publish.

Recommendation:
Add SR branch to `getCredentials` (mirror distribution worker connection lookup), skip status-check for SR when POST 200 is final, or pass `partnerUserId` on status-check job payload.

**Re-verify (f321b39):** ✅ Fixed — `getSportsRecruitsPartnerUserId` branch returns `{ partnerUserId }`.
---

---
VendorDataFetchWorker lacks SportsRecruits credentials

Risk Level: HIGH
File Path: src/workers/VendorDataFetchWorker.ts
Lines: 127-130

Description:
Same empty-credentials gap as status worker → `fetchVideoData` returns empty metadata; insight rounds won't populate for SR.

Recommendation:
Extend `getCredentials` for SR with env partner creds + linked `partnerUserId`.

**Re-verify (f321b39):** ✅ Fixed — same `getSportsRecruitsPartnerUserId` branch.
---

---
partnerUserId silent fallback to SkillShow userId

Risk Level: HIGH
File Path: src/workers/PlatformDistributionWorker.ts
Lines: 299-305

Description:
**Contract.** `partnerUserId = connectionCreds["partnerUserId"] ?? userIdForLookup` without requiring a linked SR account. Other vendors throw when connection creds are missing.

Impact:
- Publish may proceed with wrong partner user id if linking incomplete, causing SR API failures or wrong account attribution.

Recommendation:
Fail fast when `partnerUserId` is absent unless product documents SkillShow userId === SR partner user id invariant explicitly.

**Re-verify (f321b39):** ✅ Fixed — throws when `getSportsRecruitsPartnerUserId` returns undefined (no userId fallback).
---

---
No SportsRecruits unit/integration tests

Risk Level: MEDIUM
File Path: src/services/vendors/SportsRecruitsVendorService.ts
Lines: (new file)

Description:
No tests for HMAC signing, payload key order, `doPublish` error paths, or worker credential wiring after publish.

Recommendation:
Add focused tests for `auth.ts` + mocked publish; assert status-check receives credentials.

**Re-verify (f321b39):** ✅ Fixed — `tests/services/SportsRecruitsVendorService.test.ts` added (HMAC + publish paths).
---

**Positive notes:** `SportsRecruitsVendorService` extends `BaseVendorService` correctly; HMAC + key-order handling isolated in `sportsrecruits/auth.ts`; presigned URL flow aligns with `VENDORS_USING_PRESIGNED_URL`; registry and limits wired consistently.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Live secrets committed in tracked `.env.dev` | CRITICAL | Partially fixed | .env.dev | AWS 37-39; SR 73-74 |
| 2 | Post-publish status check never receives SR credentials | HIGH | ✅ Fixed | src/workers/VendorStatusCheckWorker.ts | 129-132 |
| 3 | VendorDataFetchWorker lacks SR credentials | HIGH | ✅ Fixed | src/workers/VendorDataFetchWorker.ts | 127-130 |
| 4 | partnerUserId silent fallback to SkillShow userId | HIGH | ✅ Fixed | src/workers/PlatformDistributionWorker.ts | 299-305 |
| 5 | No SportsRecruits unit/integration tests | MEDIUM | ✅ Fixed | tests/services/SportsRecruitsVendorService.test.ts | — |

**Merge readiness:** **Not merge-ready** — replace AWS keys in `.env.dev` with placeholders and rotate any exposed credentials. SR/YouTube placeholders OK. Worker wiring verified.
