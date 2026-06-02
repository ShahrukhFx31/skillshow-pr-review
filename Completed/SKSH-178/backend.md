# SKSH-178 — Backend review (`skillshow`)

**Repo:** skillshow  
**Branch:** `SKSH-178-1`  
**Base:** `main...HEAD`  
**Latest re-review:** `a517f9d` (HEAD; no newer commits on `origin/SKSH-178-1`)  
**Scope:** App users API (Critical / High only)

**Review note:** Pagination-related items are **out of scope**.

---

## Re-review summary (latest pass)

| # | Finding | Verdict |
|---|---------|---------|
| 1 | Bulk status update | **✅ Fixed** (`a517f9d`) |
| 2 | Welcome email swallowed on create | **Accepted** (team decision) |
| 3 | N+1 when resolving linked user keys on detail | **Accepted** — deferred; acceptable for typical link counts this release |

**New commits since `a517f9d`:** none (local and `origin/SKSH-178-1` match).

**New Critical/High findings:** none.

**In-scope status:** no open Critical/High findings (1 fixed, 2 accepted). **Archived:** `pr-review/Completed/SKSH-178/`.

---

## Overview

Bulk patch and repository layering improvements from `a517f9d` remain in place. All in-scope High findings are **fixed** or **accepted** for this release.

---

## Positive notes

- **`BULK_UPDATE_MAX_ITEMS` (50)** + batch `updateManyActiveStatusByIds` / `touchModificationByUserIds`.
- **`findListRowsByUserKeys`** for bulk patch and multi-key lookup infrastructure (ready for linked-user batching).
- **Layering:** athlete / relation / event / video reads via repositories in `buildDetail` / `loadLinkedEvents`.

---

## Findings

---
Welcome email failure swallowed on create

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 203

**Re-review:** **Accepted** — intentional; create succeeds when user row is created; **Resend welcome email** for recovery; failures logged via `void this.sendWelcomeEmail(..., false)`.

---

---
N+1 list aggregations when resolving linked user keys on detail

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 403-460

Description:
`loadLinkedUsers` still calls `resolveLinkedUserKey` per relation. Each call runs `appUserRepository.findListRowByUserKey` (full `listPipeline` aggregation).

```403:410:skillshow/src/services/app-user.service.ts
  private async resolveLinkedUserKey(
    linkedUserMongoId: Types.ObjectId
  ): Promise<string> {
    const row = await appUserRepository.findListRowByUserKey(
      linkedUserMongoId.toString()
    );
    return row?.userId ?? linkedUserMongoId.toString();
  }
```

Impact:
- Detail endpoint cost grows with linked-user count
- Slow loads for highly connected accounts

Recommendation:
Collect distinct linked mongo ids, call `findListRowsByUserKeys` once, build a `Map<mongoUserId, userId>`, then map relations in memory.

**PR comment (line 430):** **High:** Still one `findListRowByUserKey` aggregation per linked user—please batch via `findListRowsByUserKeys`.

**Re-review (latest):** **Accepted** — Team decision: per-link `findListRowByUserKey` is acceptable for expected linked-user volumes; batch optimization (`findListRowsByUserKeys`) may follow if metrics require it.

---

---
Bulk status update: unbounded `userIds` and sequential full patches

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 291-317

**Re-review:** **✅ Fixed** in `a517f9d`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Welcome email failure swallowed on create | HIGH | Accepted | src/services/app-user.service.ts | 203 |
| 2 | N+1 list aggregations when resolving linked user keys on detail | HIGH | Accepted | src/services/app-user.service.ts | 403-460 |
| 3 | Bulk status update: unbounded `userIds` and sequential full patches | HIGH | ✅ Fixed | src/services/app-user.service.ts | 291-317 |

### Out of scope (pagination — not reported)

| Title | Status |
|--------|--------|
| Unbounded app users list / full aggregation | Accepted — deferred |
| Activity summary vs `pagination.total` | Accepted — deferred |
| Activity video payload size / client table paging | Accepted — deferred |

---

## Merge readiness

**Complete** — all in-scope findings are **Fixed** or **Accepted**; no **Open** rows. Archived to `pr-review/Completed/SKSH-178/`.
