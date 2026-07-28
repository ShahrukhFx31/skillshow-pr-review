# Backend PR Review — skillshow (`SKSH-434`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/252  
**Branch:** `SKSH-434`  
**Base:** `main`  
**Head:** `63c9ec0eac99a129112dbf15f294a1b7c8b0d374`  
**Scope:** Public share eligibility for SkillShow / Video Library uploads — `VIDEO_PUBLIC_SHARE_QUERY`, public profile video filter via `buildMyVideosListScopeQuery`, username on share-token user lookup, swagger + tests  
**Prompts:** `pr-review/prompts/backend-system-prompt.md` (+ security policy)

**Findings:** 2 (0 Critical, 2 High) — **2 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### DRY / KISS / Global consistency / Security scan

| Check | Verdict |
|-------|---------|
| Profile grid reuses `buildMyVideosListScopeQuery` (My Videos parity) | ✅ Good DRY |
| Username selected on public profile share-token lean | ✅ Needed for SkillShow tags |
| `VIDEO_PUBLIC_SHARE_QUERY` aliased to `VIDEO_PUBLIC_ONLY_QUERY` | ❌ Collapses parent-visibility flag with unauthenticated web share; drops library visibility guards |
| `findPublicShareLean` / share profile resolution for library `user` | ❌ Admin uploader profile, not tagged athlete |
| Repository/service tests updated for profile filter | ✅ Partial (no share-by-id / soft-delete cases) |

### Positive notes

- Public profile listing correctly unions athlete + SkillShow scope instead of only `user: userId`.
- Types and share-token select updated together for `username`.

---

## GitHub comments

### `src/constants/video.constants.ts`
- **HIGH** — Public share query drops Video Library visibility guards (line 13)

### `src/swagger/video.swagger.ts`
- **HIGH** — Library public share resolves admin `user` profile, not tagged athlete (line 456)

---

## Findings

---
Public share query drops Video Library visibility guards

Risk Level: HIGH  
**Status:** Open  
File Path: src/constants/video.constants.ts  
Lines: 12-13

Description:
**Security / Global consistency.** `VIDEO_PUBLIC_SHARE_QUERY` is now aliased to `VIDEO_PUBLIC_ONLY_QUERY` (`{ isPublic: true }`), removing `inVideoLibrary: { $ne: true }`. The public profile grid correctly scopes SkillShow rows through `buildMyVideosListScopeQuery` → `VIDEO_SKILL_SHOW_TAGGED_MATCH` (excludes soft-deleted / failed library rows). Unauthenticated `findPublicShareLean` and `incrementShareViewCount` only spread `VIDEO_PUBLIC_SHARE_QUERY`, so soft-deleted (`libraryStatus: "deleted"`) or failed library documents that remain `isPublic: true` stay reachable by video ID on `/api/public/videos/:id`. This also equates the parent/coach Media Vault flag with internet share eligibility — the two were intentionally separated in SKSH-322.

Impact:
- Soft-deleted library metadata remains publicly fetchable; share view counts can still increment after admin soft-delete (S3 may already be gone).
- Profile grid vs single-video share diverge for deleted/failed library assets.
- Broader unauthenticated surface than My Videos visibility this PR claims to align with.

Recommendation:
Keep `isPublic: true` but do **not** equate share eligibility with `VIDEO_PUBLIC_ONLY_QUERY`. Encode SkillShow-capable share eligibility with library visibility guards, e.g. `$or` of athlete-upload scope and `VIDEO_SKILL_SHOW_TAGGED_MATCH` / athlete-owned `VIDEO_SKILLSHOW_UPLOADED_MATCH` (mirror `buildMyVideosListScopeQuery` clauses without requiring a username for by-id lookup, or at minimum require `libraryStatus` / `uploadStatus` guards when `inVideoLibrary: true`). Add a repository test: deleted library + `isPublic: true` → `findPublicShareLean` returns null.
---

---
Library public share resolves admin uploader profile, not tagged athlete

Risk Level: HIGH  
**Status:** Open  
File Path: src/swagger/video.swagger.ts  
Lines: 455-456  

(Blast radius: `src/services/video-public.service.ts` ~38-40; lean select in `src/constants/video.constants.ts` 15-16)

Description:
**Global consistency / Contract.** Expanding share eligibility to Video Library rows makes `GET /api/public/videos/:id` succeed for `inVideoLibrary: true` documents. Those rows set `user` to the **admin uploader**, not the tagged athlete (`libraryAthleteUsernames`). `VideoPublicService.getPublicShareView` still loads profile via `row.user`, so the public share page shows the wrong (staff) profile. Profile share listing is fine (token owner), but deep-links from that grid to `/share/video/:id` break the athlete-facing contract the swagger now advertises.

Impact:
- SkillShow public share pages show admin/uploader identity instead of the athlete.
- Possible unintended exposure of staff profile fields on an unauthenticated surface.
- Feature incomplete for the stated SkillShow share goal.

Recommendation:
For library rows, resolve the acting athlete before loading the public profile (reuse `resolveVideoActingOwnerUserId` / tagged-username resolution). Expand `VIDEO_PUBLIC_SHARE_LEAN_SELECT` with `inVideoLibrary`, `libraryAthleteUsernames` (and any fields the helper needs). When multiple athletes are tagged, define product behavior (404, primary tag, or omit profile) and test it. Add a `video-public.service` test for a library public row asserting athlete profile, not uploader.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Public share query drops Video Library visibility guards | HIGH | Open | src/constants/video.constants.ts | 12-13 |
| 2 | Library public share resolves admin uploader profile, not tagged athlete | HIGH | Open | src/swagger/video.swagger.ts | 455-456 |

**Merge readiness:** **Request changes** — 2 open High findings (security / incorrect public share profile for SkillShow library videos).
