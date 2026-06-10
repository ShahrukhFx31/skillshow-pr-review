# Backend PR Review — skillshow (`SKSH-320`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-320`  
**Base:** `main...HEAD` @ `01429de`  
**Initial review:** 2026-06-09  
**Re-reviewed:** 2026-06-09  
**Scope:** Centralized Redis pub/sub adapter for Socket.IO multi-container scaling (`@socket.io/redis-adapter`); startup ping validation; graceful shutdown for pub/sub clients (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 1 (0 Critical, 1 High) — **0 Open**, **2 Fixed**, **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `audit-log.utils.ts`, `change-stream.utils.ts` | **Not modified** |

No list endpoints, bulk row ops, audit logs, or change streams in this diff.

### Files reviewed

| File | Change | Lines (PR diff) |
|------|--------|-----------------|
| `package.json` | Add `@socket.io/redis-adapter` ^8.3.0 | 29 |
| `package-lock.json` | Lockfile for adapter | 13, 2201-2243, 8323-8328 |
| `src/config/app.ts` | `SOCKET_REDIS_ADAPTER_ENABLED` env + `config.socket` | 58-61, 164-166 |
| `src/config/redis.ts` | Pub/sub singleton; fail-fast ping; `disconnectSocketIoRedis` | 7-8, 86-153 |
| `src/config/socket.ts` | Redis adapter attach; async `createSocketServer` | 2-6, 8-30 |
| `src/app.ts` | `connectRedis()` at boot; `await createSocketServer()`; `await io.close()` on shutdown | 24, 167-169, 175-176, 193 |
| `README.md` | Socket.IO scaling section + link to scaling doc | 90-92 |
| `tests/config/redis.config.test.ts` | Ping failure, pub/sub, disconnect tests | 7, 15, 27, 56-104 |
| `tests/config/socket.config.test.ts` | Adapter on/off; async server creation | 7-22, 26-98 |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Pub/sub clients created once via `createSocketIoPubSubClients` singleton; reused by `connectRedis` + `createSocketServer` | ✅ DRY |
| Dedicated pub/sub pair separate from BullMQ/cache main client | ✅ Correct adapter pattern |
| `quitRedisClient` extracted for main + socket clients | ✅ DRY |
| Shutdown order: `await io.close()` → HTTP close → `disconnectRedis()` (pub/sub then main) | ✅ Fixed @ `01429de` |
| Protected modules untouched | ✅ |
| List/bulk/audit contracts | N/A |

### Positive notes

- **Architecture:** `@socket.io/redis-adapter` on dedicated `pub`/`sub` IORedis clients (`pub.duplicate()`) matches Socket.IO v4 guidance; existing `io.to(room).emit(...)` call sites need no changes.
- **Fail-fast startup:** `connectRedis()` pings main (+ pub/sub when adapter enabled) before boot (`src/config/redis.ts:114-123`).
- **Config toggle:** `SOCKET_REDIS_ADAPTER_ENABLED` defaults `true` for staging/prod multi-replica (`src/config/app.ts:58-61`); set `false` for single-node in-memory adapter.
- **Shutdown fix:** `src/app.ts:193` now `await io.close()` before `disconnectRedis()` — adapter teardown completes before pub/sub quit (`src/config/redis.ts:146-147`).
- **Tests:** Good coverage for adapter attach/skip, ping failure, pub/sub singleton, and disconnect paths.

---

## GitHub comments

No open Critical or High findings — no inline comments required.

**Accepted (`README.md:92`):** Broken link to `docs/socket-io-scaling.md` acknowledged; scaling runbook deferred / documented elsewhere — accepted by team.

---

## Findings

---
README links to missing Socket.IO scaling documentation

Risk Level: HIGH  
File Path: README.md  
Lines: 90-92

Description:
**Global consistency.** The PR adds a **Socket.IO scaling (multi-container)** section that directs readers to `docs/socket-io-scaling.md` for environment variables, AWS ALB settings, and staging verification. That doc was added in an earlier commit on this branch but **deleted** in `01429de` and is **not present** on the final diff vs `main`. The README link is therefore broken and deploy/ops guidance (ElastiCache sizing, `SOCKET_REDIS_ADAPTER_ENABLED`, ALB idle timeout, `VITE_SOCKET_URL`) is missing from the repo.

```90:92:skillshow/README.md
## Socket.IO scaling (multi-container)

When running multiple API containers behind a load balancer, enable the Redis adapter so WebSocket rooms and emits work across instances. See [docs/socket-io-scaling.md](docs/socket-io-scaling.md) for environment variables, AWS ALB settings, and staging verification.
```

Impact:
- Broken documentation link in the primary README for a new multi-container capability.
- Staging/prod operators lack checked-in runbook for Redis adapter env vars and ALB WebSocket configuration.
- Risk of misconfigured deploys (adapter disabled by accident, ALB idle timeout too low, wrong `maxclients` sizing).

Recommendation:
Either restore `docs/socket-io-scaling.md` (preferred — content existed on branch) or replace the link with inline essentials:

- `SOCKET_REDIS_ADAPTER_ENABLED` (default `true`; set `false` for single-container local dev)
- `REDIS_HOST` / `REDIS_PORT` / `REDIS_PASSWORD`
- ALB idle timeout ≥ 3600s; target stickiness as safety net
- `VITE_SOCKET_URL` → ALB hostname, not task IP
- Staging checklist: 2+ replicas, cross-node emit verification

**GitHub comment (inline @ `README.md:92`):** Link targets `docs/socket-io-scaling.md`, which isn't in this PR — please restore the doc or inline the env/ALB guidance and fix the link.

**Accepted (2026-06-09):** Team accepts the missing scaling doc / README link as-is for this ticket; ops guidance handled outside the repo or in a follow-up.

**Status:** Accepted

---

---
`io.close()` not awaited before Redis disconnect (re-review)

Risk Level: HIGH  
File Path: src/app.ts  
Lines: 193

Description:
**Contract.** Shutdown called `io.close()` without `await` before `disconnectRedis()` quits Socket.IO pub/sub clients when the Redis adapter is attached.

**Re-review @ `01429de`:** ✅ **Fixed** — line 193 is now `if (io) await io.close();`.

```193:195:skillshow/src/app.ts
        if (io) await io.close();
        await new Promise<void>((resolve) => httpServer.close(() => resolve()));
        await disconnectRedis();
```

**Status:** ✅ Fixed

---

---
Graceful shutdown documented but not wired (re-review)

Risk Level: HIGH  
File Path: docs/socket-io-scaling.md  
Lines: 53-55

Description:
Prior review: doc claimed shutdown order but `app.ts` did not `await io.close()`.

**Re-review @ `01429de`:** ✅ **Fixed** — doc removed from final PR; `src/app.ts:193` implements correct shutdown order in code.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | README links to missing Socket.IO scaling documentation | HIGH | Accepted | README.md | 90-92 |
| 2 | `io.close()` not awaited before Redis disconnect | HIGH | ✅ Fixed | src/app.ts | 193 |
| 3 | Graceful shutdown documented but not wired | HIGH | ✅ Fixed | docs/socket-io-scaling.md | 53-55 |

**Merge readiness:** **Merge-ready** — no open Critical/High blockers. All findings Fixed or Accepted. Core adapter wiring, startup validation, shutdown ordering (`src/app.ts:193`), and tests are in good shape.
