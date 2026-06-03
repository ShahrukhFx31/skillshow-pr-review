# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (second pass) — @ `868c05b7`  
**Scope reviewed:** All **9 files** in `git diff main...HEAD` (app-user table + partner sort alignment). Earlier SKSH-311 table pages are already on `main`.  
**Findings:** 4 prior (✅ Fixed) + **0 new** Critical/High/Medium on frontend

**Aligned with:** [backend.md](./backend.md)

### Re-verification verdict (2026-06-02)

| Area | Verdict |
|------|---------|
| Prior #1 — `listPartnersDirectory` | **✅ Fixed** — backend flat directory; client unchanged |
| Prior #2 — Partner edit/view from list page 1 | **✅ Fixed** — `getPartner` + `fetchedPartner` fallback |
| Prior #3 — Reporting managers from paginated list | **✅ Fixed** — `listSkillshowReportingManagers()` |
| Prior #4 — Filter options from current page | **✅ Fixed** — constants / `roleService` / `PARTNER_TYPE_OPTIONS` |
| **New** — App users dashboard table | **✅ OK** — `useServerTableControls`, paginated `listAppUsers`, `getAppUserRoleFilterOptions()`, server sort via `applyServerSort` |
| **New** — Partner column sort trim | **✅ OK** — removed `sorter` on `partnerType` / `status` to match backend `PARTNER_LIST_SORT_FIELD_MAP` |

No new Critical/High/Medium issues in the frontend delta. See [backend.md](./backend.md) High #3 for app-user bulk PATCH behavior (API-only).

---

---
`listPartnersDirectory` incompatible with paginated directory API

Risk Level: CRITICAL  
File Path: src/api/services/partnerService.ts  
Lines: 27-31

**Re-review:** ✅ **Fixed**

**Status:** ✅ Fixed

---

---
Partner edit/view loads partner from first list page only

Risk Level: HIGH  
File Path: src/pages/partners/onboarding/index.tsx  
Lines: 40-48

**Re-review:** ✅ **Fixed** — `getPartner(partnerId)` detail query.

**Status:** ✅ Fixed

---

---
Reporting manager options built from paginated list (10 rows)

Risk Level: HIGH  
File Path: src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx  
Lines: 106-109

**Re-review:** ✅ **Fixed** — `listSkillshowReportingManagers()`.

**Status:** ✅ Fixed

---

---
Role filter options derived from current page only

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/dashboard/index.tsx  
Lines: 59

**Re-review:** ✅ **Fixed** — `getCrewDesignationFilterOptions()`; team/partner/app-user dashboards use the same pattern.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `listPartnersDirectory` vs paginated directory API | CRITICAL | ✅ Fixed | src/api/services/partnerService.ts | 27-31 |
| 2 | Partner edit/view uses paginated list only | HIGH | ✅ Fixed | src/pages/partners/onboarding/index.tsx | 40-48 |
| 3 | Reporting managers from paginated list | HIGH | ✅ Fixed | src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx | 106-109 |
| 4 | Role/type filter options from current page | MEDIUM | ✅ Fixed | src/pages/management/crew-users/dashboard/index.tsx | 59 |

## Positive notes (full PR + app users)

- App users dashboard follows the same server-table pattern as crew/team/partners (`useServerTableControls`, meta query for empty state, `PaginationBar`).
- Sortable columns use `key` values that match backend `sortBy` (`fullName`, `createdAt`, `createdBy`); non-sortable columns (username, role, status, onboarded) have no `sorter`.
- Role filter uses `APP_USER_ROLE_OPTIONS` (not paginated rows).
- Partner table sort keys aligned with narrowed backend partner sort map.

**Merge readiness:** Frontend clear. Ticket blocked until backend High #3 (bulk status update) is fixed.