# Backend PR Review — skillshow (`SKSH-261`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-261`  
**Base:** `main...HEAD` (`d265eb6` — refactor: update username validation and suggestions)  
**Scope:** Layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical & High only)  
**Findings:** 1 (0 Critical, 1 High)

---

---
Tightened username regex blocks legacy usernames on update/check paths

Risk Level: HIGH
File Path: src/constants/username.constants.ts
Lines: 8-9

Description:
`USERNAME_REGEX` changes from `/^[a-z0-9._-]+$/` to `/^[a-z0-9]+$/`. That constant is the single source for Joi on register (`auth.validation.ts`), check-username (`username.validation.ts`), and profile username update (`profile-account-general.validation.ts`). Users who already have usernames containing `.`, `_`, or `-` in MongoDB can still log in (`loginSchema` only validates `identifier` length, not username format), but any request that re-validates the username string against the new regex will fail before `ProfileService.updateUsernameForUser` runs — including the early-return path when the submitted username equals the stored value (lines 622–624 in `profile.service.ts`), because Joi runs in route middleware first.

Impact:
- Existing accounts with legacy usernames cannot use `PATCH` profile username update (even to “keep” the same username) or pass `GET /check-username` for their current handle
- Product/support may see 400s for long-tenured users until they pick a new alphanumeric username
- No migration or grandfather rule in this PR; blast radius is all Joi consumers, not only suggestions

Recommendation:
Before merge, confirm production has zero active usernames outside `[a-z0-9]+` (or run a one-time migration). If any remain, either migrate them or allow the current username when it matches the authenticated user, e.g. custom Joi:

```ts
// In usernameUpdateSchema — illustrative; wire current username from auth context
.pattern(USERNAME_REGEX)
.when("$currentUsername", {
  is: Joi.ref("username"),
  then: Joi.string(), // skip stricter pattern when unchanged
});
```

Alternatively, validate format only in the service when `nextUsername !== user.username?.toLowerCase()`.

**PR comment (line 9):** **High:** Tightening `USERNAME_REGEX` applies to all Joi username paths, not just suggestions. Legacy usernames with `.` `_` `-` will fail profile update/check-username even when unchanged. Please confirm prod data or add a grandfather/migration path.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Tightened regex blocks legacy usernames on update/check | HIGH | Open | `src/constants/username.constants.ts` | 8-9 |

**Positive notes:** Change stays DRY — one constant drives Joi and suggestion filtering. `UsernameService.buildUsernameCandidates` aligns with `athlete.service.ts` alphanumeric sanitization. Suggestion batch availability still uses `userRepository.findTakenActiveUsernames` (`$in`, `.lean()`). Tests updated for new format and add regression coverage that suggestions match `USERNAME_REGEX`. Layering unchanged (validation → controller → service → repository).

**Not reported:** Redundant `USERNAME_HAS_ALNUM_REGEX` after the stricter primary pattern (cosmetic). `ONLY_SPECIAL_CHARACTERS` message rarely reached (test correctly expects `INVALID_FORMAT` for `.___---`). No controller/repository/query changes in diff.
