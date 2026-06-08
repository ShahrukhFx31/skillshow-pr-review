# Backend PR Review — skillshow (`SKSH-296`)

**Repo:** skillshow (main API)  
**Branch:** `SKSK-296` (remote/local; ticket id `SKSH-296`)  
**Base:** `main...HEAD` (feature commit `da2fcc6`)  
**Scope:** SkillShow user audit history — embedded `history[]`, MongoDB change streams, CDC watcher, detail API enrichment (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract enforced)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 5 (1 Critical, 4 High) — **5 Open**

> **Branch note:** Local/remote branch is named `SKSK-296`; Jira ticket is `SKSH-296`. Branch tip also contains merged `SKSH-311` commits; audit-log scope is isolated to `da2fcc6` (17 files).

---

## GitHub comments (Open findings)

### 1. `src/app.ts` lines 173-175

**Critical:** `registerSkillshowUserWatchers()` + `mongoChangeStreamService.start()` run unconditionally in every API process — with multiple replicas each change event will append duplicate `history` rows. Gate watchers to a single leader/worker role or move CDC to a dedicated consumer before production scale-out.

### 2. `src/services/mongo-change-stream.service.ts` lines 26-43

**High:** Change streams have no resume token, reconnect, or shutdown wiring — a single `error` event permanently stops audit capture for that collection until process restart, and `app.shutdown` does not close watchers.

### 3. `src/services/skillshow-user-history.watcher.ts` lines 113-117, 190-201

**High:** The `User` watcher handles every `users` collection update and then queries `findByUserIdIncludingDeleted` to filter — please narrow the stream pipeline or avoid watching the full `User` model so app/crew/athlete updates do not add DB load on every write.

### 4. `src/utils/change-stream.utils.ts` lines 61-76 / `skillshow-user-history.watcher.ts` lines 240-256

**High:** CDC mappers always set `oldValue: null`, so `buildDescription` never emits `changed from … to …` — updates are recorded as `set to` even when replacing an existing value. Enable collection pre-images or resolve prior values before persisting audit rows.

### 5. `src/utils/audit-log.utils.ts` lines 53-80 / `skillshow-user-history.watcher.ts` lines 39-54

**High:** `PendingActorStore` is in-memory per process — in multi-instance deployments actor attribution can be lost or wrong when the write and CDC flush land on different replicas. Use a shared store or synchronous history append in the service layer for actor-critical events.

---

---
Change streams run on every API replica — duplicate audit rows

Risk Level: CRITICAL  
File Path: src/app.ts  
Lines: 173-175

Description:
**Contract / reliability.** `startServer` always calls `registerSkillshowUserWatchers()` and `mongoChangeStreamService.start()`. Each API replica attaches its own `watch()` on `SkillshowUser` and `User`. MongoDB delivers the same change event to every consumer; each handler calls `pushHistoryByUserId`, so *N* replicas produce *N* identical `history` entries per write.

Impact:
- Audit log shows duplicate events for create/update/delete/resend-welcome
- KPIs or exports counting history rows are inflated
- Idempotency contract for change-stream handlers is not met (no event-id dedup)

Recommendation:
Run watchers on exactly one process (env flag / leader election), or move CDC to a dedicated worker service. Example guard:

```typescript
if (config.changeStream.enabled && config.changeStream.isLeader) {
  registerSkillshowUserWatchers();
  mongoChangeStreamService.start();
}
```

**PR comment (line 175):**  
**Critical:** `registerSkillshowUserWatchers()` + `mongoChangeStreamService.start()` run in every API process — with multiple replicas each change event will append duplicate `history` rows. Gate watchers to a single leader/worker role or move CDC to a dedicated consumer before production scale-out.

**Status:** Open

---

---
Change stream service lacks resume, reconnect, and shutdown

Risk Level: HIGH  
File Path: src/services/mongo-change-stream.service.ts  
Lines: 26-43

Description:
**Protected module / Contract.** The new `mongo-change-stream.service.ts` opens `model.watch()` and logs `stream.on("error")` but does not resume from a token, reconnect, or close streams during graceful shutdown. The module contract describes lifecycle management (watch, resume, reconnect, shutdown); `app.ts` shutdown closes HTTP/Redis but not change streams.

Impact:
- Transient Mongo/network errors permanently stop audit capture for that watcher until full process restart
- Missed changes during downtime are not backfilled
- Hot reload / deploy can strand open cursors

Recommendation:
Persist resume tokens, reconnect with backoff on `error`/`close`, and expose `mongoChangeStreamService.shutdown()` called from `app.shutdown`. Handlers should remain idempotent once duplicate-replica issue above is fixed.

**PR comment (line 39):**  
**High:** Change streams have no resume token, reconnect, or shutdown wiring — a single `error` event permanently stops audit capture for that collection until process restart, and `app.shutdown` does not close watchers.

**Status:** Open

---

---
User collection watcher processes all user updates

Risk Level: HIGH  
File Path: src/services/skillshow-user-history.watcher.ts  
Lines: 113-117, 190-201

Description:
**Performance.** `registerSkillshowUserWatchers` registers a change stream on the entire `User` model with no pipeline filter. `handleUserChange` runs on every user update in the system, then executes `findByUserIdIncludingDeleted` to see if the user is a SkillShow team member. Non-team users (app users, crew, athletes, etc.) still incur handler + DB lookup cost on every write.

Impact:
- Write amplification scales with total platform user activity, not team-user activity
- Extra load on `skillshowusers` collection for unrelated user mutations

Recommendation:
Add a `$match` pipeline stage where possible, or stop watching `User` and append user-field changes synchronously in `skillshow-user.service` (where actor context already exists). If CDC is required, consider watching only via a narrower trigger (e.g. updates tagged in service layer).

**PR comment (line 114):**  
**High:** The `User` watcher handles every `users` collection update and then queries `findByUserIdIncludingDeleted` to filter — please narrow the stream pipeline or avoid watching the full `User` model so app/crew/athlete updates do not add DB load on every write.

**Status:** Open

---

---
CDC audit rows never capture prior field values

Risk Level: HIGH  
File Path: src/utils/change-stream.utils.ts  
Lines: 61-76

Description:
**Contract / data accuracy.** `mapCdcFieldChange`, `handleUserChange` (`isActive`, `roles`), and related CDC paths always set `oldValue: null`. `buildDescription` in `audit-log.utils.ts` uses `oldValue` for `changed from … to …` clauses, but persisted rows only ever produce `set to` or `was cleared` wording — even when replacing an existing email, role, or status.

Impact:
- Admins cannot see what value was replaced in the audit UI
- Misleading copy ("Email set to new@x.com") on edits vs true creates

Recommendation:
Enable MongoDB `changeStreamPreAndPostImages` on `users` / `skillshowusers` and read pre-image fields, or capture previous values in the service before `save()` and write history synchronously for patch flows.

**PR comment (line 72):**  
**High:** CDC mappers always set `oldValue: null`, so `buildDescription` never emits `changed from … to …` — updates are recorded as `set to` even when replacing an existing value. Enable collection pre-images or resolve prior values before persisting audit rows.

**Status:** Open

---

---
In-memory pending-actor store is not replica-safe

Risk Level: HIGH  
File Path: src/utils/audit-log.utils.ts  
Lines: 53-80

Description:
**Contract / reliability.** `PendingActorStore` lives in process memory (`skillshow-user-history.watcher.ts`). `setPendingActor` / `setPendingActors` run in the HTTP handler on whichever replica serves the request, while the debounced CDC flush (1s) may execute on the same or a different replica after a load-balanced write. `take()` then misses the actor and falls back to `createdBy` or the subject user.

Impact:
- Audit rows can attribute edits to the wrong user (e.g. team member instead of admin)
- Bulk updates may lose actor context entirely under load balancing

Recommendation:
For actor-critical paths, append history synchronously in `skillshow-user.service` (actor is known), using CDC only as a safety net; or store pending actors in Redis with TTL keyed by `linkedUserId`.

**PR comment (line 53):**  
**High:** `PendingActorStore` is in-memory per process — in multi-instance deployments actor attribution can be lost or wrong when the write and CDC flush land on different replicas. Use a shared store or synchronous history append in the service layer for actor-critical events.

**Status:** Open

---

## Additional notes (not blockers)

- **Protected modules introduced:** This ticket adds `mongo-change-stream.service.ts`, `audit-log.utils.ts`, and `change-stream.utils.ts` (new on `main`). **Accepted** — ticket scope establishes shared audit/CDC infrastructure; future consumers should integrate via these modules, not ad-hoc `watch()` / inline diff logic.
- **`history` on detail only:** `getSkillshowUser` embeds full `history[]` with no pagination cap — acceptable for early rollout; monitor document size.
- **`SKILLSHOW_USER_CDC_SKIP_FIELDS` includes `history`:** Prevents feedback loop when pushing audit rows — correct.
- **`password` in `USER_CDC_SKIP_FIELDS`:** Sensitive field redaction aligned with contract.
- **Tests:** `change-stream.utils.test.ts` covers mapping helpers; no integration tests for watcher debounce/duplicate handling.
- **Typo:** `app.ts` comment `inti mongo collection watcher` → `init`.

---

## Positive notes

- Clean removal of `modificationOn` / `modificationBy` in favor of structured `history[]`.
- Service layer uses `setPendingActor` / `setPendingActors` before writes; debouncer merges rapid User + SkillshowUser updates.
- `buildDescription` + `SKILLSHOW_USER_AUDIT_DESC_OPTS` centralize copy; frontend `AuditLogTable` parses the same clause shapes.
- Detail API batches actor profile lookup via `findLeanProfileDisplayByIds`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Change streams run on every API replica — duplicate audit rows | CRITICAL | Open | src/app.ts | 173-175 |
| 2 | Change stream service lacks resume, reconnect, and shutdown | HIGH | Open | src/services/mongo-change-stream.service.ts | 26-43 |
| 3 | User collection watcher processes all user updates | HIGH | Open | src/services/skillshow-user-history.watcher.ts | 113-117, 190-201 |
| 4 | CDC audit rows never capture prior field values | HIGH | Open | src/utils/change-stream.utils.ts | 61-76 |
| 5 | In-memory pending-actor store is not replica-safe | HIGH | Open | src/utils/audit-log.utils.ts | 53-80 |

**Merge readiness:** **Not merge-ready** — 1 Critical and 4 High Open findings (multi-replica duplicate history and change-stream lifecycle must be addressed before scale-out).
