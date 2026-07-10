# PR review — FEAT-MIGRATE-S3-CLOUDFRONT (skillshow)

| Field | Value |
|-------|-------|
| PR | [#229](https://github.com/SkillshowFx/skillshow/pull/229) |
| Branch | `feat/migrate-s3-cloudfront` → `main` |
| Head | `98603914` |
| Scope | CloudFront signed GET delivery; S3 presigned GET/attachment delegation; env/config + DevOps docs |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |
| Re-reviewed | 2026-07-07 @ `98603914` |

## GitHub comments

_No open inline findings._

## Findings

---
Production may fall back to staging CloudFront private key

Risk Level: HIGH
File Path: src/utils/cloudfront-private-key.utils.ts
Lines: 1-20

Description:
**Security.** Initial review: `resolveCloudFrontPrivateKey` loaded PEM files from `cloudfront_secret/` with production fallback to `private_key-stage.pem`.

Impact:
- (Resolved) Key resolution is inline env only — no on-disk `cloudfront_secret` candidate list or cross-environment PEM fallback.

Recommendation:
✅ Fixed on `747e6e0c` / `05378995` — `resolveCloudFrontPrivateKey` normalizes `CLOUDFRONT_PRIVATE_KEY` from env only (no file-path fallback chain).

---
`cloudfront_secret` PEMs copied into `dist` without gitignore guard

Risk Level: HIGH
File Path: src/scripts/copyTemplates.ts
Lines: 8-9

Description:
**Security.** Initial review: build copied `src/cloudfront_secret/` into `dist/`, risking PEM files in images/git.

Impact:
- (Resolved) Build copies only `templates`; signing keys are expected from env/SSM (`CLOUDFRONT_PRIVATE_KEY`).

Recommendation:
✅ Fixed on `747e6e0c` — `STATIC_ASSET_DIRS = ["templates"]`; `cloudfront_secret` removed from build copy.

---
Unsigned `buildObjectUrl` requires signing private key

Risk Level: MEDIUM
File Path: src/services/cloudfront.service.ts
Lines: 35-42

Description:
Initial review: `buildObjectUrl` called `assertSigningConfig()`, requiring private key even for unsigned `location` metadata URLs.

Impact:
- (Resolved) `buildObjectUrl` now uses `assertDomainConfig()` only; signing key checks remain on `signUrl` / `getSignedGetUrl`.

Recommendation:
✅ Fixed on `959d6cc0` — split `assertDomainConfig` / `assertSigningConfig`; test `buildObjectUrl requires only CLOUDFRONT_DOMAIN` added.

---
CloudFront config only enforced in production

Risk Level: HIGH
File Path: src/config/app.ts
Lines: 304-353

Description:
Initial review / revbot: `CLOUDFRONT_*` checks lived only in production fail-fast paths while `s3Service` routed all GET URLs through CloudFront signing.

Impact:
- (Resolved) Non-production boots without CloudFront would fail on first media request instead of at startup.

Recommendation:
✅ Fixed on `05378995` — `assertStartupSecurityConfig()` calls `assertCloudFrontConfig()` in **all** environments via `startServer`; tests added in `tests/config/app.security.test.ts`.

---
Legacy stored URLs not re-signed for client display (images / recent uploads)

Risk Level: HIGH
File Path: src/utils/s3upload.ts
Lines: 43-62

Description:
**Regression.** After migration, list endpoints still returned bare S3/CloudFront metadata URLs or passed through unsigned `https://` values instead of issuing fresh signed GET URLs for browser display.

Impact:
- Thumbnails, profile images, partner logos, and recent-upload lists could 403 once OAC blocks direct S3/unsigned CloudFront reads.

Recommendation:
✅ Fixed on `05378995` — `resolveClientObjectUrl` centralizes re-signing from stored URL or explicit key; `getRecentUploads` uses it with `r.key`; `s3ClientDisplayUrlForStored` no longer passes through unsigned external URLs.

---
Attachment downloads depend on CloudFront forwarding S3 override query params

Risk Level: MEDIUM
File Path: src/services/cloudfront.service.ts
Lines: 84-97

Description:
`getSignedAttachmentUrl` embeds `response-content-disposition` as a query parameter. CloudFront must forward override query strings to the S3 origin (unlike native S3 presigned GET `ResponseContentDisposition`).

Impact:
- Valid signed URLs may render inline or use wrong filenames if distribution cache/origin policies omit override params.

Recommendation:
**Accepted (ops dependency).** `docs/cloudfront-devops.md` and `src/constants/cloudfront.constants.ts` document required forwarded query strings, cache-key behavior, and smoke-test steps.

---
Env example templates removed without full replacement

Risk Level: MEDIUM
File Path: .env.dev.example
Lines: 1

Description:
**Maintainability.** PR deletes `.env.dev.example` and `.env.prod.example`. `docs/cloudfront-devops.md` documents CloudFront/S3 vars only, not the full application env surface (MongoDB, JWT, Redis, email, OAuth, etc.).

Impact:
- New developers lose the in-repo canonical env checklist; misconfiguration risk outside the documented CloudFront slice.

Recommendation:
Restore `.env.dev.example` / `.env.prod.example` (or a single `.env.example`) with the full var list including `CLOUDFRONT_*`, or add a complete env template under `docs/` and link from the README.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Production may fall back to staging CloudFront private key | HIGH | ✅ Fixed | src/utils/cloudfront-private-key.utils.ts | 1-20 |
| 2 | `cloudfront_secret` PEMs copied into `dist` without gitignore guard | HIGH | ✅ Fixed | src/scripts/copyTemplates.ts | 8-9 |
| 3 | Unsigned `buildObjectUrl` requires signing private key | MEDIUM | ✅ Fixed | src/services/cloudfront.service.ts | 35-42 |
| 4 | CloudFront config only enforced in production | HIGH | ✅ Fixed | src/config/app.ts | 304-353 |
| 5 | Legacy stored URLs not re-signed for client display | HIGH | ✅ Fixed | src/utils/s3upload.ts | 43-62 |
| 6 | Attachment downloads depend on CloudFront query forwarding | MEDIUM | Accepted | src/services/cloudfront.service.ts | 84-97 |
| 7 | Env example templates removed without full replacement | MEDIUM | Open | .env.dev.example | 1 |

**Merge readiness:** No open Critical/High blockers. Ready to merge once `CLOUDFRONT_*` vars are set in all environments and CloudFront distribution policies match `docs/cloudfront-devops.md`. Optional follow-up: restore full env example templates (#7).
