# PR review — FEAT-MIGRATE-S3-CLOUDFRONT (skillshow)

| Field | Value |
|-------|-------|
| PR | [#229](https://github.com/SkillshowFx/skillshow/pull/229) |
| Branch | `feat/migrate-s3-cloudfront` → `main` |
| Scope | CloudFront signed GET delivery; S3 presigned GET/attachment delegation; env/config + DevOps docs |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |

## GitHub comments

### `src/utils/cloudfront-private-key.utils.ts` (line 73)

**HIGH** — Production may fall back to staging CloudFront private key

### `src/scripts/copyTemplates.ts` (line 9)

**HIGH** — `cloudfront_secret` PEMs copied into `dist` without gitignore guard

### `src/services/cloudfront.service.ts` (line 28)

**MEDIUM** — Unsigned `buildObjectUrl` requires signing private key

## Findings

---
Production may fall back to staging CloudFront private key

Risk Level: HIGH
File Path: src/utils/cloudfront-private-key.utils.ts
Lines: 72-77

Description:
**Security.** When `CLOUDFRONT_PRIVATE_KEY` / `CLOUDFRONT_PRIVATE_KEY_PATH` are empty, `resolveCloudFrontPrivateKey` loads PEM files from `cloudfront_secret/`. In production the candidate order is `[private_key-prod.pem, private_key.pem, private_key-stage.pem]`. If the prod PEM is missing, production silently falls back to the **staging** key.

Impact:
- Production signed URLs may be generated with the staging key pair while the distribution trusts the production key group — URLs fail or, worse, operators may attach the staging public key to prod without noticing.
- Cross-environment key misuse weakens key rotation and blast-radius controls.

Recommendation:
In production, load **only** `private_key-prod.pem` (or explicit `CLOUDFRONT_PRIVATE_KEY_PATH` / env PEM). If prod PEM is missing, fail fast (mirror `assertProductionSecurityConfig`) instead of trying stage/default candidates.

---
`cloudfront_secret` PEMs copied into `dist` without gitignore guard

Risk Level: HIGH
File Path: src/scripts/copyTemplates.ts
Lines: 8-9

Description:
**Security.** Build now copies `src/cloudfront_secret/` → `dist/cloudfront_secret/` alongside templates. `.gitignore` does not exclude `*.pem` under that folder. A developer placing a real signing key locally for staging tests can accidentally commit it and ship it in the production Docker image (`dist/` is included in the runtime image).

Impact:
- CloudFront signing private keys in git history or container filesystem.
- Key exposure bypasses SSM/Secrets Manager controls documented in `docs/cloudfront-devops.md`.

Recommendation:
1. Add `src/cloudfront_secret/*.pem` (or the whole directory except `.gitkeep`) to `.gitignore`.
2. Do not bundle `cloudfront_secret` into production images — production should require `CLOUDFRONT_PRIVATE_KEY` from SSM/env only.
3. Keep dev/staging PEM-on-disk as an opt-in local path, not a build artifact.

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
| 1 | Production may fall back to staging CloudFront private key | HIGH | Open | src/utils/cloudfront-private-key.utils.ts | 72-77 |
| 2 | `cloudfront_secret` PEMs copied into `dist` without gitignore guard | HIGH | Open | src/scripts/copyTemplates.ts | 8-9 |
| 3 | Unsigned `buildObjectUrl` requires signing private key | MEDIUM | Open | src/services/cloudfront.service.ts | 26-33 |

**Merge readiness:** Blocked — fix production key resolution and secret handling before cutover to CloudFront-signed delivery.
