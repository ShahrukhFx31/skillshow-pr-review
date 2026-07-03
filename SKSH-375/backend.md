# PR review — SKSH-375 (skillshow)

| Field | Value |
|-------|-------|
| PR | [#228](https://github.com/SkillshowFx/skillshow/pull/228) |
| Branch | `SKSH-375-major` → `main` |
| Head | `SKSH-375-major` @ `d1371c5e` |
| Scope | Athlete onboarding search — auto-provision missing athlete profiles; username/email-only user match; populated search rows |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |

## GitHub comments

### `src/repositories/athlete.repository.ts`
- **HIGH** — Event athlete search not updated for missing-profile auto-provision (line 249; targets `src/services/event-athlete.service.ts` ~172–180)

## Findings

---
Event athlete search not updated for missing-profile auto-provision

Risk Level: HIGH
File Path: src/services/event-athlete.service.ts
Lines: 172-180

Description:
**Global consistency.** This PR fixes parent onboarding athlete search by auto-creating minimal `Athlete` rows in `findOnboardingSearchByUserIdsPaginated` before returning results. `EventAthleteService.listAvailableForEvent` still resolves matches via `findProfileIdsByUserIds` only — athlete-role users who match username/email but have no profile document (signup before account-general) remain invisible when adding athletes to an event.

Impact:
- Same data gap the PR fixes for onboarding search persists on event registration search.
- Admins cannot find/link athletes by username/email in event flows until a profile row exists through another code path.

Recommendation:
Before `findProfileIdsByUserIds`, reuse the same batch provision path — e.g. extract the missing-profile `insertMany` block from `findOnboardingSearchByUserIdsPaginated` into a shared repository helper (`ensureAthleteProfilesForUserIds`) and call it from both `athlete.service.search` and `event-athlete.service.listAvailableForEvent`. Add a service test covering event available search when the user exists but the athlete profile row is missing.

---
Profile auto-provision logic duplicated in repository

Risk Level: MEDIUM
File Path: src/repositories/athlete.repository.ts
Lines: 154-168, 249-276 (approx.)

Description:
**DRY.** `ensureAthleteIdForUser` (single-user, RBAC-guarded, active-user check) and the new `insertMany` path inside `findOnboardingSearchByUserIdsPaginated` (batch, no role re-check) both create minimal `{ user }` athlete documents with separate duplicate-key handling.

Impact:
- Future changes to provisioning rules (active-user guard, role check, soft-delete handling) can diverge between onboarding search and other entry points (`parent-link`, `athlete-relation`, event flows).

Recommendation:
Extract one internal helper, e.g. `ensureAthleteProfilesForUserIds(userIds: Types.ObjectId[])`, used by both `ensureAthleteIdForUser` and `findOnboardingSearchByUserIdsPaginated`. Keep RBAC/active-user checks in the single-user path; batch path should still verify callers only pass athlete-role user ids (document in JSDoc).

### Positive notes

- **Root-cause fix:** Populating `user` on onboarding search rows removes the fragile `userById` map and fixes missing results when athlete profiles did not exist yet.
- **Contract alignment:** Athlete search JSDoc specifies username/email only; removing supplemental profile-text search and narrowing `usernameOrEmailContainsFilter` matches that spec.
- **Hardening:** `ensureAthleteIdForUser` now verifies the user exists and is not deleted before creating a profile.
- **Concurrency:** `insertMany({ ordered: false })` with duplicate-only error swallow handles parallel profile creation safely.
- **Types:** `AthleteOnboardingSearchRow` replaces `Record<string, unknown>` casts in the service mapper.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Event athlete search not updated for missing-profile auto-provision | HIGH | Open | src/services/event-athlete.service.ts | 172-180 |
| 2 | Profile auto-provision logic duplicated in repository | MEDIUM | Open | src/repositories/athlete.repository.ts | ~154-276 |

**Merge readiness:** Open HIGH blocker — apply the missing-profile provision fix to event athlete available search (or justify scoped exception).
