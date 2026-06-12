# Backend PR Review — skillshow (`SKSH-115`)

**Repo:** skillshow  
**Branch:** `sksh-115`  
**Base:** `main...HEAD` @ `3bef25e`  
**Re-verified:** `3bef25e` (`fix: feedbacks`)  
**Scope:** Admin dashboard KPI/auth changes (Critical / High / Medium)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 3 in scope (1 Critical, 0 High, 2 Medium) — **3 fixed**, **0 open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Cosmetic only** — whitespace/formatting; no behavioral change |
| All other protected modules | **Not modified** ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/routes/super-admin.routes.ts` | Split read auth (`admin` + `super_admin`) vs cache flush (`super_admin` only) |
| `src/services/super-admin.service.ts` | KPI counts: `countUsersByRoleIds` + `eventRepository.countActiveListingEvents()` |
| `src/repositories/event.repository.ts` | Added `countActiveListingEvents()` using `activeListingMatch()` |
| `src/repositories/super-admin.repository.ts` | Removed unused `countActiveEvents()` |
| `src/validation/list-query.validation.ts` | Formatting-only |
| `package-lock.json` | Incidental `libc` optional metadata removal |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| KPI counts | ✅ Restored role-based user count; events via dedicated `countActiveListingEvents` |
| Event count location | ✅ KISS — count lives on `eventRepository` with shared `activeListingMatch()` |
| Auth split read vs flush | ✅ `superAdminDashboardReadAuth` vs `superAdminOnlyAuth` |
| Protected list-query module | ✅ No contract change |

### Positive notes

- **Events KPI:** `countActiveListingEvents()` uses `activeListingMatch()` (handles `listingStatus` + legacy `status`), better aligned with event list APIs than the removed `countActiveEvents()` (`status` only).
- **Auth:** Cache flush correctly restricted to `super_admin` while dashboard read is shared with platform `admin`.

---

## GitHub comments

No open Critical or High findings — prior inline comments resolved on `3bef25e`.

---

## Findings

---
KPI totals use listRows with wrong filters (regression)

Risk Level: CRITICAL  
**Status:** ✅ Fixed  
File Path: src/services/super-admin.service.ts  
Lines: 136-152

Description:
Initial diff replaced dedicated counts with `appUserRepository.listRows` / `eventRepository.listRows`, changing crew-inclusive user totals and active-only event totals.

**Re-verification (`3bef25e`):** ✅ **Fixed** — `buildKpis` restores `superAdminRepository.countUsersByRoleIds(appUserRoleIds)` and uses `eventRepository.countActiveListingEvents()`.

**PR comment:** **Resolved** — KPI semantics restored.

---

---
listRows used as expensive count-only shortcut

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/services/super-admin.service.ts  
Lines: 140-145

Description:
Initial diff used `listRows` solely to read `.total`, running full paginated aggregates.

**Re-verification (`3bef25e`):** ✅ **Fixed** — dedicated count methods only; no `listRows` in `buildKpis`.

**PR comment:** **Resolved** — count-only path removed.

---

---
Admin role authorized to flush super-admin dashboard cache

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/routes/super-admin.routes.ts  
Lines: 17-31

Description:
Initial diff allowed `admin` on both `GET /dashboard` and `DELETE /dashboard/cache`.

**Re-verification (`3bef25e`):** ✅ **Fixed** — `superAdminDashboardReadAuth` (`admin` + `super_admin`) for `GET`; `superAdminOnlyAuth` (`super_admin` only) for `DELETE /dashboard/cache`.

**PR comment:** **Resolved** — auth split correctly.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | KPI totals use listRows with wrong filters (regression) | CRITICAL | ✅ Fixed | src/services/super-admin.service.ts | 136-152 |
| 2 | listRows used as expensive count-only shortcut | MEDIUM | ✅ Fixed | src/services/super-admin.service.ts | 140-145 |
| 3 | Admin role authorized to flush super-admin dashboard cache | MEDIUM | ✅ Fixed | src/routes/super-admin.routes.ts | 27-31 |

**Merge readiness:** No open Critical/High/Medium blockers. All findings fixed on `3bef25e`. Ready to merge.
