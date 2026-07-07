# PR review — FEAT-MIGRATE-S3-CLOUDFRONT (skillshow)

| Field | Value |
|-------|-------|
| PR | [#229](https://github.com/SkillshowFx/skillshow/pull/229) |
| Branch | `feat/migrate-s3-cloudfront` → `main` |
| Head | `747e6e0c` |
| Scope | CloudFront signed GET delivery; S3 presigned GET/attachment delegation; env/config + DevOps docs |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |
| Re-reviewed | 2026-07-07 @ `747e6e0c` |

## GitHub comments

### `src/services/cloudfront.service.ts` (line 28)

**MEDIUM** — Unsigned `buildObjectUrl` requires signing private key

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
Lines: 26-33

Description:
`buildObjectUrl` (used by `s3Service.getPublicUrl` for stored `location` metadata on upload record endpoints) calls `assertSigningConfig()`, which requires domain, key pair id, **and** private key. Constructing an unsigned distribution URL only needs `CLOUDFRONT_DOMAIN`.

Impact:
- Upload record / video create flows fail unless full signing material is present, even though the returned URL is not signed.
- Harder local/staging bootstrap when only the distribution domain is configured.

Recommendation:
Split config checks: `assertDomainConfigured()` for `buildObjectUrl`; keep full signing assertions on `signUrl` / `getSignedGetUrl` only.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Production may fall back to staging CloudFront private key | HIGH | ✅ Fixed | src/utils/cloudfront-private-key.utils.ts | 27-39 |
| 2 | `cloudfront_secret` PEMs copied into `dist` without gitignore guard | HIGH | ✅ Fixed | src/scripts/copyTemplates.ts | 8-9 |
| 3 | Unsigned `buildObjectUrl` requires signing private key | MEDIUM | Open | src/services/cloudfront.service.ts | 26-33 |

**Merge readiness:** No open Critical/High blockers. One Medium follow-up on `buildObjectUrl` domain-only config check (optional before merge if env always supplies full signing material).
