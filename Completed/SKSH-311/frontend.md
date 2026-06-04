# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full pr-verify) — @ `5c37a009`  
**Scope reviewed:** **Full PR diff** — all **27 files** in `git diff main...HEAD` per frontend system prompt.  
**Findings:** 5 prior + **0 new** Critical/High/Medium — all **✅ Fixed**

**Aligned with:** [backend.md](./backend.md)

### Full review coverage

| Area | Verdict |
|------|---------|
| Prior #1 — `listPartnersDirectory` | ✅ Fixed |
| Prior #2 — Partner edit/view | ✅ Fixed — `getPartner` |
| Prior #3 — Reporting managers | ✅ Fixed — `listSkillshowReportingManagers()` |
| Prior #4 — Filter options from page rows | ✅ Fixed — static/constants helpers |
| Prior #5 — Video library bulk status N× PATCH | ✅ Fixed — `bulkUpdateVideoLibrary` only |
| App users dashboard (server table, bulk drawer) | ✅ OK |
| Video library page/table (server table, bulk) | ✅ OK |
| Partners dashboard + `partners-bulk-update.tsx` | ✅ OK — `bulkPatchPartners` single API |
| Partner column sort trim | ✅ OK |

### pr-verify counts

| Metric | Count |
|--------|-------|
| New findings added | 0 |
| Prior Open → Fixed this pass | 0 (already fixed on branch) |
| Remaining Open | 0 |

---

---
`listPartnersDirectory` incompatible with paginated directory API

Risk Level: CRITICAL  
File Path: src/api/services/partnerService.ts  
Lines: 32-36

**Full re-review:** ✅ **Fixed** — `listPartnersDirectory(): Promise<PartnerRow[]>` against flat `/directory`.

**Status:** ✅ Fixed

---

---
Partner edit/view loads partner from first list page only

Risk Level: HIGH  
File Path: src/pages/partners/onboarding/index.tsx  
Lines: 40-48

**Full re-review:** ✅ **Fixed** — `getPartner(partnerId)` detail query.

**Status:** ✅ Fixed

---

---
Reporting manager options built from paginated list (10 rows)

Risk Level: HIGH  
File Path: src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx  
Lines: 106-109

**Full re-review:** ✅ **Fixed** — `listSkillshowReportingManagers()`.

**Status:** ✅ Fixed

---

---
Role filter options derived from current page only

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/dashboard/index.tsx  
Lines: 59

**Full re-review:** ✅ **Fixed** — `getCrewDesignationFilterOptions()`; app users use `getAppUserRoleFilterOptions()`.

**Status:** ✅ Fixed

---

---
Video library bulk status fires one PATCH per selected row

Risk Level: HIGH  
File Path: src/pages/videoLibrary/dashboard/components/video-library-table.tsx  
Lines: 68-73

**Full re-review:** ✅ **Fixed** — `bulkUpdateMutation` always calls `bulkUpdateVideoLibrary({ videoIds, ... })`; no `statusOnly` / `Promise.all` / per-row `patchVideoLibraryItem` branch.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `listPartnersDirectory` vs paginated directory API | CRITICAL | ✅ Fixed | src/api/services/partnerService.ts | 32-36 |
| 2 | Partner edit/view uses paginated list only | HIGH | ✅ Fixed | src/pages/partners/onboarding/index.tsx | 40-48 |
| 3 | Reporting managers from paginated list | HIGH | ✅ Fixed | src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx | 106-109 |
| 4 | Role/type filter options from current page | MEDIUM | ✅ Fixed | src/pages/management/crew-users/dashboard/index.tsx | 59 |
| 5 | Video library bulk status uses N× PATCH instead of bulk API | HIGH | ✅ Fixed | src/pages/videoLibrary/dashboard/components/video-library-table.tsx | 68-73 |

## Positive notes (full diff)

- Server-table pattern consistent on app users, video library, and partners (`useServerTableControls`, meta queries, `PaginationBar`).
- Partner bulk UI uses `bulkPatchPartners` (matches new backend batched route).
- App-user bulk UI uses single `bulkPatchAppUsers` (backend batched path verified in [backend.md](./backend.md) #3).

**Merge readiness:** ✅ Clear — archived 2026-06-02.
