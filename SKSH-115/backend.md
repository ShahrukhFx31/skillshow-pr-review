# Backend PR Review — skillshow (`SKSH-115`)

**Repo:** skillshow  
**Branch:** `sksh-115`  
**Base:** `main...HEAD` @ `08a09c9`  
**Scope:** Admin dashboard KPI/auth changes; super-admin service KPI sourcing (Critical / High / Medium)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 3 in scope (1 Critical, 0 High, 2 Medium) — **3 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Cosmetic only** — whitespace/formatting on `listQueryOptionalLimitField`; no behavioral change |
| All other protected modules | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/routes/super-admin.routes.ts` | Allow `admin` role on dashboard + cache flush routes |
| `src/services/super-admin.service.ts` | KPI counts via `appUserRepository.listRows` / `eventRepository.listRows` |
| `src/validation/list-query.validation.ts` | Formatting-only |
| `package-lock.json` | Incidental `libc` optional metadata removal |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| KPI counts reuse list repositories | ⚠️ Semantics diverge from prior dedicated count methods (see #1) |
| `listRows` used only for `.total` | ⚠️ KISS / performance — full list aggregate for a count (see #2) |
| Auth expansion for `admin` role | ⚠️ Intentional for admin dashboard; cache flush scope needs confirmation (see #3) |
| Protected list-query module | ✅ No contract change |

---

## GitHub comments (Open findings)

### 1. `src/services/super-admin.service.ts` line 140

**PR comment (line 140):** **Critical:** `buildKpis` now sources `totalAppUsers` and `events` from `listRows({ ...DEFAULT_LIST_QUERY })` (lines 140, 143), which changes KPI semantics. Previously `countUsersByRoleIds` included **crew**; `appUserRepository.listRows` only surfaces athlete/parent/coach. Previously `countActiveEvents` filtered `status: ACTIVE`; `listRows` without `status` counts all non-deleted events. Restore original filters or pass equivalent query params (e.g. `status: EVENT_STATUS.ACTIVE` for events).

### 2. `src/services/super-admin.service.ts` line 140

**PR comment (line 140):** **Medium:** Using `listRows` solely to read `.total` (lines 140–143, 146–149) runs a full paginated aggregate. Prefer dedicated `countDocuments` / count helpers with the same match filters, or a count-only repository method.

### 3. `src/routes/super-admin.routes.ts` line 29

**PR comment (line 29):** **Medium:** `DELETE /dashboard/cache` now uses `superAdminDashboardAuth` (line 18: `admin` + `super_admin`). Confirm admins should flush shared cache — if read-only access is enough, restrict `DELETE` to `super_admin` only.

---

## Findings

---
KPI totals use listRows with wrong filters (regression)

Risk Level: CRITICAL  
File Path: src/services/super-admin.service.ts  
Lines: 137-151

Description:
**Contract / behavioral regression.** `buildKpis` replaced purpose-built count methods with `appUserRepository.listRows` and `eventRepository.listRows` using bare `DEFAULT_LIST_QUERY`.

- **Total App Users:** Was `countUsersByRoleIds` over `SUPER_ADMIN_APP_USER_ROLE_NAMES` (athlete, parent, coach, **crew**) on the `users` collection. New path uses the app-user list aggregate, which only includes `APP_USER_RBAC_ROLE_NAMES` (athlete, parent, coach — **excludes crew**) and applies a different join/match pipeline.
- **Events:** Was `countActiveEvents()` (`isDeleted: false`, `status: EVENT_STATUS.ACTIVE`). New path calls `eventRepository.listRows` with **no** `status` filter, so draft/inactive/archived events are included in the KPI total.

Impact:
- Admin and super-admin dashboards show incorrect KPI numbers
- KPI "Events" can disagree with the admin dashboard events preview card (which filters `status: active` on the frontend)
- Crew user count drops out of "Total App Users" if the prior metric was intentional

Recommendation:
Restore semantic parity. Either revert to `superAdminRepository.countUsersByRoleIds` / `countActiveEvents`, or pass explicit filters to `listRows`:

```typescript
appUserRepository.listRows({ ...DEFAULT_LIST_QUERY, role: /* if supported, or keep dedicated count */ }),
eventRepository.listRows({ ...DEFAULT_LIST_QUERY, status: EVENT_STATUS.ACTIVE }),
```

If the new definitions are intentional, document them in the ticket and align frontend labels (e.g. "All Events" vs "Active Events").

**PR comment (line 140):** **Critical:** `buildKpis` `listRows` totals (lines 140, 143) no longer match the old crew-inclusive app-user count or active-only event count — KPIs will be wrong on admin/super-admin dashboards.

---

---
listRows used as expensive count-only shortcut

Risk Level: MEDIUM  
File Path: src/services/super-admin.service.ts  
Lines: 137-151

Description:
**KISS / performance.** `buildKpis` only needs aggregate totals but calls `listRows`, which runs `runListQueryAggregate` (match, sort, `$facet` with data + count) and materializes up to `DEFAULT_PAGE_SIZE` (10) row projections per call.

Impact:
- Unnecessary DB work on every dashboard load (cached, but still heavier than `countDocuments`)
- Sets a precedent of misusing list endpoints for KPI counts

Recommendation:
Add or reuse lightweight count methods with the same match filters, e.g. `EventModel.countDocuments({ isDeleted: false, status: EVENT_STATUS.ACTIVE })` and the existing role-based user count, or a `countRows(query)` on repositories that skips the data facet.

**PR comment (line 140):** **Medium:** `listRows` for KPI `.total` only (lines 140–143) is heavier than a count — use dedicated count helpers with the same filters.

---

---
Admin role authorized to flush super-admin dashboard cache

Risk Level: MEDIUM  
File Path: src/routes/super-admin.routes.ts  
Lines: 18, 27-31

Description:
**Scope / auth.** `superAdminDashboardAuth` now allows both `super_admin` and `admin` on `GET /dashboard` and `DELETE /dashboard/cache`. Read access aligns with the new admin dashboard UI, but cache flush is a destructive/shared operation previously limited to `super_admin`.

Impact:
- Platform admins can invalidate cached chart/KPI data for all consumers
- May be intentional; if not, widens operational privilege unexpectedly

Recommendation:
If only read access is required for admins, split middleware: shared read auth for `GET /dashboard`, `super_admin`-only for `DELETE /dashboard/cache`.

**PR comment (line 29):** **Medium:** Confirm admins should flush super-admin dashboard cache (`superAdminDashboardAuth` at line 18), or restrict `DELETE` (lines 27–31) to `super_admin` only.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | KPI totals use listRows with wrong filters (regression) | CRITICAL | Open | src/services/super-admin.service.ts | 140, 143 |
| 2 | listRows used as expensive count-only shortcut | MEDIUM | Open | src/services/super-admin.service.ts | 140-143 |
| 3 | Admin role authorized to flush super-admin dashboard cache | MEDIUM | Open | src/routes/super-admin.routes.ts | 29 |

**Merge readiness:** **Not merge-ready** — 1 open Critical (KPI count regression). Fix #1 before merge; address #2–#3 or mark Accepted with ticket justification.
