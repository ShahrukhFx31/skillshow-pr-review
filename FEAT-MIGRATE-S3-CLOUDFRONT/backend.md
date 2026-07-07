# PR review — FEAT-MIGRATE-S3-CLOUDFRONT (skillshow)

| Field | Value |
|-------|-------|
| PR | [#229](https://github.com/SkillshowFx/skillshow/pull/229) |
| Branch | `feat/migrate-s3-cloudfront` → `main` |
| Head | `959d6cc0` |
| Scope | CloudFront signed GET delivery; S3 presigned GET/attachment delegation; env/config + DevOps docs |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |
| Re-reviewed | 2026-07-07 @ `959d6cc0` |

## GitHub comments

_No open inline findings._

## Findings

---
Production may fall back to staging CloudFront private key

Risk Level: HIGH
File Path: src/utils/cloudfront-private-key.utils.ts
Lines: 27-39

Description:
**Security.** Initial review: `resolveCloudFrontPrivateKey` loaded PEM files from `cloudfront_secret/` with production fallback to `private_key-stage.pem`.

Impact:
- (Resolved) Key resolution is now env/path only — no on-disk `cloudfront_secret` candidate list or cross-environment PEM fallback.

Recommendation:
✅ Fixed on `747e6e0c` — `resolveCloudFrontPrivateKey` returns inline `CLOUDFRONT_PRIVATE_KEY` or reads `CLOUDFRONT_PRIVATE_KEY_PATH` only; production fail-fast remains in `assertProductionSecurityConfig`.

---
`cloudfront_secret` PEMs copied into `dist` without gitignore guard

Risk Level: HIGH
File Path: src/scripts/copyTemplates.ts
Lines: 8-9

Description:
**Security.** Initial review: build copied `src/cloudfront_secret/` into `dist/`, risking PEM files in images/git.

Impact:
- (Resolved) Build copies only `templates`; signing keys are expected from env/SSM (`CLOUDFRONT_PRIVATE_KEY` / `CLOUDFRONT_PRIVATE_KEY_PATH`).

Recommendation:
✅ Fixed on `747e6e0c` — `STATIC_ASSET_DIRS = ["templates"]`; `cloudfront_secret` removed from build copy and DevOps docs.

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

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Production may fall back to staging CloudFront private key | HIGH | ✅ Fixed | src/utils/cloudfront-private-key.utils.ts | 27-39 |
| 2 | `cloudfront_secret` PEMs copied into `dist` without gitignore guard | HIGH | ✅ Fixed | src/scripts/copyTemplates.ts | 8-9 |
| 3 | Unsigned `buildObjectUrl` requires signing private key | MEDIUM | ✅ Fixed | src/services/cloudfront.service.ts | 35-42 |

**Merge readiness:** No open Critical/High/Medium blockers. CloudFront migration ready for OAC cutover once `CLOUDFRONT_*` env vars are set in staging/production.
