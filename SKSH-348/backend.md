# Backend PR Review — skillshow (`SKSH-348`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-348`  
**Base:** `main...HEAD` @ `e665d29`  
**Initial review:** 2026-06-16  
**Scope:** Parent linked-athlete teams list (`GET /api/v1/athletes/:relationId/teams`) — coach teams where the athlete is a member (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules / Security)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 0 (0 Critical, 0 High) — **0 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| Route auth: `linkedAthleteProfileGuards("read")` (relation + RBAC) | ✅ Matches sibling `complete-profile` / `account-general` |
| **IDOR:** `athleteId` from `req.relationAccess` after accepted-relation check | ✅ No raw athlete/user ID from caller |
| Relation middleware: parent/coach role + `findAcceptedByIdForUserRelation` | ✅ |
| Team soft-delete: `TeamModel` pre-`find` hook filters `isDeleted: false` | ✅ |
| Coach display names: `findLeanDisplayWithEmailByIds` excludes deleted users | ✅ |
| Logo URLs: existing `presignGetUrlOrNull` via `mapCoachTeamListItem` | ✅ |
| No commented/weakened `authorize`, upload, or export paths touched | ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/controllers/athlete.controller.ts` | `listLinkedTeams` handler |
| `src/routes/athlete.routes.ts` | `GET /:relationId/teams` route |
| `src/services/athlete.service.ts` | `listLinkedTeams` → owner userId + DTO map |
| `src/services/coach-link.service.ts` | `listMemberAthleteTeamsWithCoachNames`, `loadDisplayNamesByUserIds` |
| `src/repositories/coach-link.repository.ts` | `findTeamsByMemberAthleteIdLean` |
| `src/types/athlete.type.ts` | `LinkedAthleteTeamItem` / `LinkedAthleteTeamsResponse` |
| `src/types/coach-link.types.ts` | `coachUserId` on `LeanCoachTeamListRow` |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuses `linkedAthleteProfileGuards` + `resolveOwnerUserIdForAthlete` (same as other linked-athlete reads) | ✅ |
| `loadDisplayNamesByUserIds` batches coach name lookup (no per-team user query) | ✅ DRY |
| `mapCoachTeamListItem` + `findTeamsByMemberAthleteIdLean` mirror existing coach team list patterns | ✅ |
| `displayNameFromUser` null-safe email (supports coach rows) | ✅ |
| Non-paginated list acceptable for bounded athlete team membership | ✅ Accepted |
| Parent-only UI on frontend; API allows coach per existing linked-athlete read contract | ✅ Accepted (consistent with `getCompleteProfile`) |

### Positive notes

- Thin controller → service → repository flow; types live in `src/types/`.
- Cross-coach team discovery is correctly keyed off the athlete **user** id (`memberAthleteIds`), not relation id.
- `Promise.all` + batched coach names avoids N+1 user fetches (S3 presign per team matches existing `listTeams` pattern).

---

## GitHub comments

_No Critical or High findings._

---

## Findings

_No Critical or High findings._

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** ✅ No Critical or High blockers on the backend. Safe to merge from a backend review perspective.
