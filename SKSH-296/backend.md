# Backend PR Review — skillshow (`SKSH-296`)

**Repo:** skillshow (main API)  
**Branch:** `SKSK-296` (remote/local; ticket id `SKSH-296`)  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-08 @ `42adbc7`  
**Scope:** SkillShow user audit history — embedded `history[]`, MongoDB change streams, CDC watcher, detail API enrichment (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract enforced)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 5 (0 Critical, 0 High open) — **3 Fixed**, **2 Accepted**

> **Branch note:** Local/remote branch is named `SKSK-296`; Jira ticket is `SKSH-296`. Audit-log core: `da2fcc6`; bug-fix pass: `42adbc7` (leader gating, Redis actors, stream lifecycle).

### Protected modules

This PR **introduces** `mongo-change-stream.service.ts`, `audit-log.utils.ts`, and `change-stream.utils.ts` (new on `main`). **Accepted** for ticket scope — future consumers must integrate via these modules.

---

---

---
Change streams gated to leader replica

Risk Level: CRITICAL  
File Path: src/services/change-stream.bootstrap.ts  
Lines: 7-20

Description:
**Contract / reliability.** `startChangeStreamConsumersIfLeader()` skips registration unless `CHANGE_STREAM_ENABLED` and `CHANGE_STREAM_LEADER` are true. `app.ts` calls this instead of unconditionally starting watchers; shutdown invokes `stopChangeStreamConsumers()` → `mongoChangeStreamService.shutdown()`.

Impact:
- Multi-replica duplicate `history` rows avoided when ops sets `CHANGE_STREAM_LEADER=false` on non-leader replicas

Recommendation:
N/A — implemented. Document deploy requirement: exactly one replica with `CHANGE_STREAM_LEADER=true`.

**Re-review (`42adbc7`):** ✅ **Fixed** — `change-stream.bootstrap.ts` lines 7-20; `app.ts` line 174 (`startChangeStreamConsumersIfLeader`), line 200 (`stopChangeStreamConsumers`).

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Change stream service resume, reconnect, and shutdown

Risk Level: HIGH  
File Path: src/services/mongo-change-stream.service.ts  
Lines: 47-130

Description:
**Protected module / Contract.** Service now stores `resumeToken`, reconnects with exponential backoff on `close`, and exposes `shutdown()` that clears timers and closes streams.

Impact:
- Transient stream errors no longer permanently stop audit capture until process restart
- Graceful deploy closes change-stream cursors

Recommendation:
N/A — implemented.

**Re-review (`42adbc7`):** ✅ **Fixed** — `resumeAfter` at lines 79-81; `shutdown` 47-68; reconnect 97-130.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
User collection watcher processes all user updates

Risk Level: HIGH  
File Path: src/services/skillshow-user-history.watcher.ts  
Lines: 99-103, 173-184

Description:
**Performance.** `registerSkillshowUserWatchers` still registers a change stream on the entire `User` model with no pipeline filter. `handleUserChange` runs on every user update, then executes `findByUserIdIncludingDeleted` to see if the user is a SkillShow team member.

Impact:
- Write amplification scales with total platform user activity, not team-user activity
- Extra load on `skillshowusers` collection for unrelated user mutations

Recommendation:
Add a `$match` pipeline stage where possible, or stop watching `User` and append user-field changes synchronously in `skillshow-user.service` (actor context already exists).

**PR comment (`skillshow-user-history.watcher.ts` line 100):** Accepted — full-`User` stream + post-filter is intentional for v1; optimize in a follow-up if write volume warrants it.

**Status:** Accepted

**Accepted:** Team accepts the tradeoff: watcher listens to all `users` updates and filters to SkillShow team members in-process. Acceptable for initial rollout; revisit with pipeline narrowing or synchronous service writes if load becomes an issue.

---

---
CDC audit rows never capture prior field values

Risk Level: HIGH  
File Path: src/utils/change-stream.utils.ts  
Lines: 68-86

Description:
**Contract / data accuracy.** `mapCdcFieldChange` and `handleUserChange` (`isActive`, `roles` at watcher lines 210-224) always set `oldValue: null`. `buildDescription` supports `changed from … to …`, but persisted rows only produce `set to` or `was cleared` wording on edits.

Impact:
- Admins cannot see what value was replaced in the audit UI
- Misleading copy ("Email set to new@x.com") on edits vs true creates

Recommendation:
Enable MongoDB `changeStreamPreAndPostImages` on `users` / `skillshowusers` and read pre-image fields, or capture previous values in the service before `save()` and write history synchronously for patch flows.

**PR comment (line 80):** Accepted — `oldValue` omitted for v1; audit rows use `set to` / `was cleared` wording until pre-images or synchronous capture is added.

**Status:** Accepted

**Accepted:** Team accepts missing prior values on CDC-driven updates for initial release. Pre-images or service-layer capture can be a follow-up when richer diff copy is required.

---

---
Pending-actor store moved to Redis

Risk Level: HIGH  
File Path: src/utils/audit-log.utils.ts  
Lines: 68-121

Description:
**Contract / reliability.** `PendingActorStore` now persists actors in Redis (`SET` / `GETDEL` with TTL) so any API replica can write and the CDC leader can read the actor on flush.

Impact:
- Actor attribution works across load-balanced replicas when Redis is available

Recommendation:
N/A — implemented. Ensure Redis is reachable in all environments running CDC.

**Re-review (`42adbc7`):** ✅ **Fixed** — Redis-backed `PendingActorStore`; async `setPendingActor` / `setPendingActors` in watcher.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

## Additional notes (not blockers)

- **Deploy:** `CHANGE_STREAM_LEADER` defaults `true` — multi-replica setups must set `CHANGE_STREAM_LEADER=false` on all but one replica or duplicates return.
- **`history` on detail only:** No pagination cap on embedded array — acceptable for early rollout.
- **`SKILLSHOW_USER_CDC_SKIP_FIELDS` includes `history`:** Prevents CDC feedback loop — correct.
- **`password` in `USER_CDC_SKIP_FIELDS`:** Sensitive field redaction aligned with contract.
- **Tests:** `change-stream.utils.test.ts` covers mapping helpers; no integration tests for leader gating or Redis actor round-trip.

---

## Positive notes

- Clean removal of `modificationOn` / `modificationBy` in favor of structured `history[]`.
- Leader bootstrap + stream lifecycle match protected-module contract intent.
- Redis-backed actor stash is the right fix for multi-replica writes.
- Service layer uses `setPendingActor` / `setPendingActors` before writes; debouncer merges rapid User + SkillshowUser updates.
- `buildDescription` + `SKILLSHOW_USER_AUDIT_DESC_OPTS` centralize copy; frontend `AuditLog` parses the same clause shapes.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Change streams gated to leader replica | CRITICAL | ✅ Fixed | src/services/change-stream.bootstrap.ts | 7-20 |
| 2 | Change stream service resume, reconnect, and shutdown | HIGH | ✅ Fixed | src/services/mongo-change-stream.service.ts | 47-130 |
| 3 | User collection watcher processes all user updates | HIGH | Accepted | src/services/skillshow-user-history.watcher.ts | 99-103, 173-184 |
| 4 | CDC audit rows never capture prior field values | HIGH | Accepted | src/utils/change-stream.utils.ts | 68-86 |
| 5 | Pending-actor store moved to Redis | HIGH | ✅ Fixed | src/utils/audit-log.utils.ts | 68-121 |

**Merge readiness:** **Merge-ready (backend)** — all findings Fixed or Accepted. Deploy: one replica with `CHANGE_STREAM_LEADER=true`, Redis for pending actors.
