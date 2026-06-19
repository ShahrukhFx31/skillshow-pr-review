# Backend PR Review — skillshow (`SKSH-359`)

**Repo:** skillshow — `git@github-work:SkillshowFx/skillshow.git`  
**Branch:** `SKSH-359`  
**Base:** `main...HEAD` @ `72cb2f2`  
**Initial review:** 2026-06-19  
**Scope:** Username collision retry for user creation, import-tool parallel chunk persist, soft-deleted duplicate detection (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules / Security)

**Findings:** 0 (0 Critical, 0 High) — **0 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| No route, middleware, auth, upload, or export changes | ✅ |
| Import row-key paths still require `actorUserId` (`401` when missing) | ✅ |
| No weakened `authorize`, IDOR, or S3 paths touched | ✅ |
| User creation still flows through existing `authRepository.createUser` / `userRepository.createAthleteUser` guards | ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/username.constants.ts` | `USERNAME_GENERATION_MAX_ATTEMPTS`, `USERNAME_COLLISION_MAX_RETRIES` |
| `src/constants/import-tool.constants.ts` | Doc update — chunk rows persisted concurrently |
| `src/repositories/import-tool.repository.ts` | `findExistingEmails` / `findExistingUsernames` include soft-deleted users (aligns with unique indexes) |
| `src/services/username.service.ts` | `pickUniqueUsername`, `runWithUsernameCollisionRetry`, `runWithFixedUsernameRetry` |
| `src/utils/username.utils.ts` | `isUsernameCollisionError` |
| `src/utils/import-tool-import.utils.ts` | `createEmptyImportChunkAccumulator`, `foldImportRowOutcomes` |
| `src/utils/import-tool.utils.ts` | `USERNAME_ALREADY_TAKEN` row-error mapping |
| `src/services/athlete.service.ts` | Delegates username generation + insert retry to `usernameService` |
| `src/services/skillshow-user.service.ts` | Same collision-retry wrapper for admin SkillShow user create |
| `src/services/import-tool/import-tool-row-importer.service.ts` | Parallel chunk persist (`Promise.all`), coach/parent fixed-username retry, athlete import emails non-blocking |
| Tests | `username.service`, `username.utils`, `import-tool-import.utils`, athlete/skillshow-user mocks updated |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Removed duplicate `generateUniqueUsername` from `athlete.service.ts`; single source in `username.service.ts` | ✅ DRY |
| Import chunk accumulator extracted to `import-tool-import.utils.ts` | ✅ DRY |
| `runWithUsernameCollisionRetry` shared by athlete create, SkillShow user create, and import athlete rows | ✅ Reusability |
| `runWithFixedUsernameRetry` for coach/parent CSV usernames with shared collision detection | ✅ |
| Import validation duplicate checks now match Mongo unique indexes (soft-deleted rows included) | ✅ Global consistency |
| Parallel chunk import paired with insert-time collision retry for auto-generated usernames | ✅ |
| `crew-user.service.ts` / `app-user.service.ts` still use `pickFirstAvailableUsername` without insert retry | ✅ Accepted — not in PR diff; optional follow-up if those paths see parallel creation pressure |
| `existsActiveUsername` / `findTakenActiveUsernames` still scope to active users while unique index is global | ✅ Accepted — `isUsernameCollisionError` + retry handles soft-deleted holder races; import validation now rejects soft-deleted fixed usernames/emails |

### Positive notes

- `isUsernameCollisionError` distinguishes username vs email duplicate-key errors — email failures do not trigger username retry loops.
- `pickUniqueUsername` re-checks `existsActiveUsername` per candidate after batch filter, reducing parallel-import check-then-insert races before insert.
- Unit tests cover collision retry, max-retry exhaustion, non-username error passthrough, and fixed-username retry.
- Athlete import sets `awaitPostCreateEmails: false`, matching fire-and-forget coach/parent welcome email show pattern and avoiding email backpressure blocking parallel chunk completion.

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

**Merge readiness:** ✅ No Critical or High blockers on the backend. Username generation is centralized with insert-time collision retry; import duplicate detection aligns with Mongo unique indexes; parallel chunk import is appropriately paired with retry logic.
