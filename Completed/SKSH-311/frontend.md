# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 — @ `e9b0ef55` (`fix: bug fixes`)  
**Scope:** Server-side tables for partners, crew users, and SkillShow team users (Critical, High, Medium)  
**Findings:** 4 (0 Critical, 0 High, 0 Medium open) — **all prior findings fixed**

**Aligned with:** [backend.md](./backend.md)

### Re-verification verdict (2026-06-02)

| # | Original issue | Verdict |
|---|----------------|---------|
| 1 | `listPartnersDirectory` vs paginated directory | **✅ Fixed** — backend restored flat `PartnerRow[]`; client `listPartnersDirectory()` unchanged and correct |
| 2 | Partner edit/view from first list page only | **✅ Fixed** — `getPartner(partnerId)` + `fetchedPartner` fallback (same pattern as team users) |
| 3 | Reporting managers from paginated list | **✅ Fixed** — `listSkillshowReportingManagers()` via `/reporting-managers` |
| 4 | Role/type filters from current page | **✅ Fixed** — crew: `CREW_USER_DESIGNATION_OPTIONS`; team: `TEAM_USER_DEPARTMENT_OPTIONS` + `roleService`; partners: `PARTNER_TYPE_OPTIONS` |

No new Critical/High/Medium issues identified in the post-fix diff.

---

---
`listPartnersDirectory` incompatible with paginated directory API

Risk Level: CRITICAL  
File Path: src/api/services/partnerService.ts  
Lines: 27-31

Description:
(Initial review.) Directory returned `{ items, pagination }` while client expected `PartnerRow[]`.

**Re-review:** ✅ **Fixed** — backend `directory` handler returns a flat array again; `listPartnersDirectory()` remains `PartnerRow[]` and Connect Social `.filter` works.

**PR comment (line 22):** **Critical:** Directory now returns `{ items, pagination }` … *(resolved on backend + client contract restored.)*

---

---
Partner edit/view loads partner from first list page only

Risk Level: HIGH  
File Path: src/pages/partners/onboarding/index.tsx  
Lines: 40-48

Description:
(Initial review.) Edit/view resolved partner only from `listPartners` page 1.

**Re-review:** ✅ **Fixed** — `getPartner(partnerId)` query with `fetchedPartner` fallback in `selectedPartner` memo.

**PR comment (line 35):** **High:** Edit/view still resolve the partner from `listPartners` with default page size 10 … *(resolved with by-id fetch.)*

---

---
Reporting manager options built from paginated list (10 rows)

Risk Level: HIGH  
File Path: src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx  
Lines: 106-109

Description:
(Initial review.) Dropdown used `listSkillshowUsers({ ...DEFAULT_LIST_QUERY })` (10 rows).

**Re-review:** ✅ **Fixed** — `useQuery({ queryFn: () => listSkillshowReportingManagers(), ... })`.

**PR comment (line 104):** **High:** Reporting manager dropdown is built from the paginated list … *(resolved — dedicated endpoint.)*

---

---
Role filter options derived from current page only

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/dashboard/index.tsx  
Lines: 59

Description:
(Initial review.) `buildRoleFilterOptions(crewUsers)` used paginated `data.items`.

**Re-review:** ✅ **Fixed** — `getCrewDesignationFilterOptions()` from `CREW_USER_DESIGNATION_OPTIONS`; team dashboard uses `getDepartmentFilterOptions()` and role options from `roleService`; partners use `getPartnerTypeFilterOptions()` from `PARTNER_TYPE_OPTIONS`.

**PR comment (line 55):** **Medium:** `crewUsers` is now `data.items` but filter options still treat it like a full list … *(resolved — constants / role service.)*

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `listPartnersDirectory` vs paginated directory API | CRITICAL | ✅ Fixed | src/api/services/partnerService.ts | 27-31 |
| 2 | Partner edit/view uses paginated list only | HIGH | ✅ Fixed | src/pages/partners/onboarding/index.tsx | 40-48 |
| 3 | Reporting managers from paginated list | HIGH | ✅ Fixed | src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx | 106-109 |
| 4 | Role/type filter options from current page | MEDIUM | ✅ Fixed | src/pages/management/crew-users/dashboard/index.tsx | 59 |

## Positive notes

- `useServerTableControls` centralizes debounced search, sort, and page reset on filter changes.
- Column `key` / `dataIndex` align with backend `sortBy` (`fullName`, `createdAt`, `crewDesignation`).
- Filter dropdowns use stable enums/APIs instead of the current table page.
- Partner onboarding mirrors team-user detail loading (`getPartner` + list cache).

**Merge readiness:** Frontend clear — no open Critical/High/Medium on `SKSH-311` @ `e9b0ef55`. Ready to merge with backend.
