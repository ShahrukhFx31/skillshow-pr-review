# Backend PR Review — skillshow (`SKSH-273`)

**Repo:** skillshow  
**Branch:** `SKSH-273`  
**Base:** `main...HEAD`  
**Scope:** Layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical and High only)  
**Findings:** 1 (0 Critical, 1 High)

---

---
Public share treats `isPublic` as “not false” while upload UI treats only `true` as public

Risk Level: HIGH
File Path: src/utils/video-share.utils.ts
Lines: 5-7

Description:
`isVideoPubliclyShareable` returns `true` when `isPublic` is `undefined` or omitted (`isPublic !== false`). The video schema defaults `isPublic` to `true`, and `mapVideoToResponse` / `getPublicShareView` use this helper to expose `shareUrl` and serve the public endpoint. The admin upload flow (SKSH-273) now treats only `isPublic === true` as public and sends `isPublic: false` on draft save when the switch is off—but videos that still have `undefined` or default `true` in Mongo remain publicly reachable at `/api/public/videos/:id` even when the athlete believes the toggle is off.

Impact:
- Athletes can think a video is private (switch off, no share field in UI) while the share URL and presigned `playUrl` still work for anyone with the link
- Inconsistent contract between API list payloads (`shareUrl` present) and UI visibility rules

Recommendation:
Align server rules with the product rule “explicitly public only”, e.g.:

```ts
export function isVideoPubliclyShareable(isPublic?: boolean | null): boolean {
  return isPublic === true;
}
```

Consider defaulting `isPublic` to `false` on new uploads (schema migration or set on create) so legacy rows are handled explicitly. Update tests in `tests/utils/video-share.utils.test.ts` that currently expect `undefined` to be shareable.

**PR comment (line 6):**  
**High:** `isVideoPubliclyShareable` treats missing/`undefined` `isPublic` as public while the new upload UI only enables sharing when `isPublic === true`. Please align backend share eligibility with explicit `true` (and revisit the model default) so private-by-UI videos cannot be opened via `/api/public/videos/:id`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Public share treats `isPublic` as “not false” vs UI `=== true` | HIGH | Open | `src/utils/video-share.utils.ts` | 5-7 |

**Positive notes:** Clean layering (routes → controller → `VideoPublicService` → repository). Params validated with `videoIdParamSchema`. Public route correctly sits outside `/v1` auth. `VIDEO_PUBLIC_SHARE_QUERY` excludes library videos. Profile subset via `toPublicVideoShareProfile` strips coach emails, nickname, and DOB. Good unit coverage for share utils, profile mapping, and service. Swagger documents the new endpoint.

**Skipped (per prompt):** Medium issues (e.g. silent `catch` in `getPublicVideoShareProfileForUser`, view-count inflation from bots, separate find + increment round-trips).

**Re-review update (latest `SKSH-273`):** Finding re-verified and still reproducible; no additional backend Critical/High issues were introduced in the new role-based profile mapping commit.

**Merge readiness:** Blocked — 1 open High finding.
