# Backend PR Review — skillshow (`SKSH-320`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-320`  
**Base:** `main...HEAD` @ `a7a8732`  
**Initial review:** 2026-06-09  
**Revised:** 2026-06-09 (line refs added for all findings and GitHub comments)  
**Scope:** Centralized Redis pub/sub adapter for Socket.IO multi-container scaling (`@socket.io/redis-adapter`); startup ping validation; graceful shutdown for pub/sub clients (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 2 (0 Critical, 2 High) — **2 Open**

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
| `src/app.ts` | `connectRedis()` at boot; `await createSocketServer()` | 24, 167-169, 175-176 |
| `docs/socket-io-scaling.md` | ALB, env, staging checklist, startup/shutdown | 1-55 (new) |
| `README.md` | Link to scaling doc | 90-93 |
| `tests/config/redis.config.test.ts` | Ping failure, pub/sub, disconnect tests | 7, 15, 27, 56-104 |
| `tests/config/socket.config.test.ts` | Adapter on/off; async server creation | 7-22, 26-98 |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Pub/sub clients created once via `createSocketIoPubSubClients` singleton; reused by `connectRedis` + `createSocketServer` | ✅ DRY |
| Dedicated pub/sub pair separate from BullMQ/cache main client | ✅ Correct adapter pattern |
| `quitRedisClient` extracted for main + socket clients | ✅ DRY |
| `createSocketServer` marked `async` with no `await` inside | Minor KISS — harmless |
| Protected modules untouched | ✅ |
| List/bulk/audit contracts | N/A |

### Positive notes

- **Architecture:** `@socket.io/redis-adapter` on dedicated `pub`/`sub` IORedis clients (`pub.duplicate()`) matches Socket.IO v4 guidance; existing `io.to(room).emit(...)` call sites need no changes.
- **Fail-fast startup:** `connectRedis()` now pings main (+ pub/sub when adapter enabled) before boot; documented in `docs/socket-io-scaling.md:49-51`.
- **Config toggle:** `SOCKET_REDIS_ADAPTER_ENABLED` defaults `true` for staging/prod multi-replica (`src/config/app.ts:58-61`); can disable for single-node in-memory adapter.
- **Tests:** Good coverage for adapter attach/skip, ping failure, pub/sub singleton, and disconnect paths.
- **Docs:** Practical ALB idle timeout, stickiness, ElastiCache `maxclients` sizing, and staging verification checklist.

---

## GitHub comments

| File | Lines | Comment |
|------|-------|---------|
| `docs/socket-io-scaling.md` | 55 | This line documents ordered graceful shutdown (Socket.IO → HTTP → pub/sub Redis → main Redis), and the PR adds `disconnectSocketIoRedis()` (`src/config/redis.ts:136-147`), but `src/app.ts:193-195` still calls `io.close()` without `await` before `disconnectRedis()`. Please update shutdown in this PR so the documented order is enforced. |
| `src/app.ts` | 176 | `await createSocketServer(httpServer)` now attaches the Redis adapter (`src/config/socket.ts:20-23`). The shutdown handler at `src/app.ts:190-195` still runs un-awaited `io.close()` before `disconnectRedis()` quits pub/sub clients (`src/config/redis.ts:146-147`). Please `await io.close()` at line 193 before `disconnectRedis()`. |

---

## Findings

---
Graceful shutdown documented but not wired in PR

Risk Level: HIGH  
File Path: docs/socket-io-scaling.md  
Lines: 53-55

**Related lines:** `src/config/redis.ts:136-147` (`disconnectSocketIoRedis` / `disconnectRedis`); `src/app.ts:190-195` (shutdown handler — fix target)

Description:
**Global consistency / Contract.** This PR adds a **Graceful shutdown** section claiming SIGTERM/SIGINT closes Socket.IO, HTTP, Socket.IO pub/sub clients, then main Redis—in that order. The diff adds pub/sub teardown in `src/config/redis.ts:136-147` but does not update `src/app.ts:193-195` to `await io.close()` before `disconnectRedis()`. With the Redis adapter enabled (`src/config/socket.ts:20-23`), pub/sub clients may be quit while Socket.IO is still closing.

```53:55:skillshow/docs/socket-io-scaling.md
## Graceful shutdown

On `SIGTERM` / `SIGINT` the server closes Socket.IO, the HTTP server, Socket.IO Redis pub/sub clients, then the main Redis connection—in that order.
```

```136:147:skillshow/src/config/redis.ts
export async function disconnectSocketIoRedis(): Promise<void> {
  await Promise.all([
    quitRedisClient(socketPubClient),
    quitRedisClient(socketSubClient),
  ]);
  socketPubClient = null;
  socketSubClient = null;
  logger.info("redis.disconnectSocketIoRedis: done");
}

export async function disconnectRedis(): Promise<void> {
  await disconnectSocketIoRedis();
```

```190:195:skillshow/src/app.ts
    const shutdown = async (signal: string) => {
      try {
        logger.info("app.shutdown: starting", { signal });
        if (io) io.close();
        await new Promise<void>((resolve) => httpServer.close(() => resolve()));
        await disconnectRedis();
```

Impact:
- ElastiCache pub/sub connections may leak or error during ECS task rotation.
- Documented shutdown contract (`docs/socket-io-scaling.md:55`) does not match runtime behavior.
- Staging verification step 5 (`docs/socket-io-scaling.md:46`) may surface adapter/Redis errors in logs.

Recommendation:
At `src/app.ts:193`, await Socket.IO close before Redis disconnect:

```typescript
if (io) await io.close();
await new Promise<void>((resolve) => httpServer.close(() => resolve()));
await disconnectRedis();
```

**GitHub comment (inline @ `docs/socket-io-scaling.md:55`):** This section documents shutdown order including pub/sub teardown, but `src/app.ts:193-195` isn't updated to `await io.close()` before `disconnectRedis()`. Please wire the documented sequence in this PR.

---

---
Redis adapter attached without awaited Socket.IO close on shutdown

Risk Level: HIGH  
File Path: src/app.ts  
Lines: 176, 193-195

**Related lines:** `src/config/socket.ts:20-23` (adapter attach); `src/config/redis.ts:146-147` (`disconnectRedis` → pub/sub quit); `docs/socket-io-scaling.md:55` (documented contract)

Description:
**Contract.** Line 176 changes Socket.IO init to `io = await createSocketServer(httpServer)`, which attaches `@socket.io/redis-adapter` when enabled (`src/config/socket.ts:20-23`). The shutdown handler at lines 193-195 (unchanged in diff) still runs `if (io) io.close()` without `await`, then `await disconnectRedis()`—which quits the dedicated pub/sub pair at `src/config/redis.ts:146-147`. Adapter teardown must finish before pub/sub `quit()`.

```175:176:skillshow/src/app.ts
    // Initialize Socket.IO using centralized config (Redis adapter when enabled)
    io = await createSocketServer(httpServer);
```

```20:23:skillshow/src/config/socket.ts
  if (config.socket.redisAdapterEnabled) {
    const { pub, sub } = createSocketIoPubSubClients();
    io.adapter(createAdapter(pub, sub));
    logger.info("socket.io: Redis adapter attached");
```

```193:195:skillshow/src/app.ts
        if (io) io.close();
        await new Promise<void>((resolve) => httpServer.close(() => resolve()));
        await disconnectRedis();
```

Impact:
- Race between adapter close and pub/sub disconnect on deploy/restart.
- Conflicts with `docs/socket-io-scaling.md:55`.

Recommendation:
Change `src/app.ts:193` to `if (io) await io.close();` before `disconnectRedis()` at line 195.

**GitHub comment (inline @ `src/app.ts:176`):** Now that the Redis adapter is attached here, please `await io.close()` at `src/app.ts:193` before `disconnectRedis()` quits the pub/sub clients.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Graceful shutdown documented but not wired in PR | HIGH | Open | docs/socket-io-scaling.md | 53-55 |
| 2 | Redis adapter attached without awaited Socket.IO close on shutdown | HIGH | Open | src/app.ts | 176, 193-195 |

**Merge readiness:** **Not merge-ready** — 2 open High findings. Core adapter wiring, config, and tests are solid; update `src/app.ts:193-195` (`await io.close()` before `disconnectRedis()`) so it matches pub/sub teardown at `src/config/redis.ts:146-147` and the contract at `docs/socket-io-scaling.md:55`.
