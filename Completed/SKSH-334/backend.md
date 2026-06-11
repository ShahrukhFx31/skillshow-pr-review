# Backend PR Review — skillshow (`SKSH-334`)

**Repo:** skillshow (main API) — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-334`  
**Base:** `main...HEAD` @ `43d4ffe`  
**Initial review:** 2026-06-10  
**Re-reviewed:** 2026-06-10 (`43d4ffe` — merge-only since prior review; no new code findings)  
**Scope:** Relax platform event assignment on video PATCH — allow self-upload without athlete profile; coach/parent self-uploads; share-path permission guard only when `viewerUserId !== ownerUserId` (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 0 (0 Critical, 0 High)

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `change-stream.utils.ts`, `audit-log.*` | **Not modified** |

No list endpoints, bulk row ops, audit logs, or change streams in this diff.

### Files reviewed

| File | Change | Lines (PR diff) |
|------|--------|-----------------|
| `src/services/event-athlete.service.ts` | Skip profile/permission gate on self-upload; use `ensureAthleteIdForUser` on share path | 6, 113-127 |
| `tests/services/event-athlete.service.test.ts` | Integration tests for athlete/coach/parent self-upload assignment | 119-213 |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuses `ensureAthleteIdForUser` (existing athlete-repo helper) instead of ad-hoc profile lookup | ✅ DRY |
| Self-upload path skips redundant `assertViewerMayAssignPlatformEvent` (method already no-ops when viewer === owner) | ✅ KISS |
| Share path still enforces `assertViewerMayAssignPlatformEvent` + `canShareLinkedAthleteVideos` | ✅ Contract |
| `resolveVideoActingOwnerUserId` unchanged; owner resolution consistent with library tagging | ✅ |
| Protected modules untouched | ✅ |
| List/bulk/audit contracts | N/A |

### Positive notes

- **Behavior fix:** Athlete-role users can assign a platform event on their own upload before completing account-general (no athlete row required). Test asserts profile stays `null` after assignment (`event-athlete.service.test.ts:176-177`).
- **Role coverage:** Coach and parent self-uploads now succeed; previously blocked by mandatory `findIdByUserId` profile check.
- **Share path:** Cross-user assignment still requires athlete profile (auto-provisioned via `ensureAthleteIdForUser` when owner has athlete role) and share permission; non-athlete owners on share path get `403` with `EVENT_PLATFORM_VIDEO_SHARE_FORBIDDEN_MESSAGE`.
- **Tests:** Two focused integration cases cover the ticket’s core scenarios.
- **Optional cleanup (out of scope):** `EVENT_PLATFORM_ATHLETE_PROFILE_REQUIRED_MESSAGE` in `event.constants.ts` is now unused; safe to remove in a follow-up.

---

## GitHub comments

No Critical or High findings — no inline comments required.

---

## Findings

*No Critical or High findings.*

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** No open Critical/High blockers. Safe to merge from a backend perspective.
