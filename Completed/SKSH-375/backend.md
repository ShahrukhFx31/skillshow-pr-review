# PR review — SKSH-375 (skillshow)

| Field | Value |
|-------|-------|
| PR | [#228](https://github.com/SkillshowFx/skillshow/pull/228) |
| Branch | `SKSH-375-major` → `main` |
| Head | `SKSH-375-major` @ `8292624a` |
| Scope | Athlete onboarding + event search — auto-provision missing athlete profiles; username/email-only user match; populated search rows |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |
| Re-reviewed | 2026-07-03 @ `8292624a` |

## GitHub comments

_No open inline findings._

## Findings

---
Event athlete search not updated for missing-profile auto-provision

Risk Level: HIGH
File Path: src/services/event-athlete.service.ts
Lines: 178-182

Description:
**Global consistency.** Initial review: `listAvailableForEvent` used `findProfileIdsByUserIds` only, so athlete-role users without profile rows were invisible in event add-athlete search. Re-review: calls `ensureAthleteProfilesForUserIds` before resolving profile ids.

Impact:
- (Resolved) Event and onboarding search now share the same missing-profile provision path.

Recommendation:
✅ Fixed in `8292624a` — keep `ensureAthleteProfilesForUserIds(matchingUserIds)` before `findProfileIdsByUserIds`.

---
Profile auto-provision logic duplicated in repository

Risk Level: MEDIUM
File Path: src/repositories/athlete.repository.ts
Lines: 161-165, 237-272, 286

Description:
**DRY.** Initial review: batch `insertMany` in `findOnboardingSearchByUserIdsPaginated` duplicated `ensureAthleteIdForUser` logic. Re-review: extracted `ensureAthleteProfilesForUserIds`; used by `ensureAthleteIdForUser`, `findOnboardingSearchByUserIdsPaginated`, and `event-athlete.service.listAvailableForEvent`.

Impact:
- (Resolved) Single source of truth for minimal athlete row creation.

Recommendation:
✅ Fixed as implemented.

### Positive notes

- **DRY:** `ensureAthleteProfilesForUserIds` centralizes active-user check, `insertMany`, and duplicate-key handling.
- **Global consistency:** Onboarding search (`findOnboardingSearchByUserIdsPaginated`) and event available search both provision before lookup.
- **RBAC preserved:** `ensureAthleteIdForUser` still gates on athlete role before delegating to the shared helper.
- **Contract alignment:** Athlete search remains username/email only; populated `AthleteOnboardingSearchRow` removes fragile `userById` map.

### Optional follow-up (not reported as findings)

- Add `event-athlete.service` test for available search when user exists but profile row is missing (mirrors athlete search coverage).

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Event athlete search not updated for missing-profile auto-provision | HIGH | ✅ Fixed | src/services/event-athlete.service.ts | 178-182 |
| 2 | Profile auto-provision logic duplicated in repository | MEDIUM | ✅ Fixed | src/repositories/athlete.repository.ts | 161-165, 237-286 |

**Merge readiness:** No open Critical/High/Medium blockers.
