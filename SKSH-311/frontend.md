# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full review) — @ `e38016e1`  
**Scope reviewed:** All **24 files** in `git diff main...HEAD` (app-users dashboard server table; video library server table + columns/hooks; partner sort alignment; shared table types).  
**Findings:** 4 prior (✅ Fixed) + **1 new High** (video library bulk status)

**Aligned with:** [backend.md](./backend.md)

### Review coverage (full pass)

| Area | Verdict |
|------|---------|
| Prior #1 — `listPartnersDirectory` | **✅ Fixed** — flat directory API; client unchanged |
| Prior #2 — Partner edit/view from list page 1 | **✅ Fixed** — `getPartner` + `fetchedPartner` fallback |
| Prior #3 — Reporting managers from paginated list | **✅ Fixed** — `listSkillshowReportingManagers()` |
| Prior #4 — Filter options from current page | **✅ Fixed** — constants / `roleService` / `PARTNER_TYPE_OPTIONS` |
| App users dashboard | **✅ OK** — `useServerTableControls`, paginated `listAppUsers`, `getAppUserRoleFilterOptions()`, bulk drawer calls `bulkPatchAppUsers` (backend regression tracked separately) |
| Video library page/table | **⚠️ High #5** — status-only bulk bypasses `bulkUpdateVideoLibrary` |
| Partner column sort trim | **✅ OK** — removed `sorter` on `partnerType` / `status` to match backend sort map |

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

---
Video library bulk status fires one PATCH per selected row

Risk Level: HIGH  
File Path: src/pages/videoLibrary/dashboard/components/video-library-table.tsx  
Lines: 71-73

Description:
When bulk update is status-only, `bulkUpdateMutation` uses `Promise.all(videoIds.map((id) => patchVideoLibraryItem(id, { status })))` instead of `bulkUpdateVideoLibrary({ videoIds, status })`. Joi allows up to **200** video ids on `PATCH /bulk`, but this path issues up to 200 separate HTTP requests (each hitting single-item patch logic on the server).

Impact:
- Large selections can overwhelm the browser connection pool and API (many concurrent PATCHs).
- Partial failures are harder to surface than the bulk endpoint’s `failed[]` response.
- Inconsistent with event/athlete bulk paths in the same mutation, which correctly call `bulkUpdateVideoLibrary`.

Recommendation:
Use the bulk endpoint for status-only updates as well:

```ts
if (statusOnly) {
  return bulkUpdateVideoLibrary({ videoIds, status });
}
```

**PR inline comment:** GitHub only allows comments on **changed** diff lines. The `statusOnly` / `Promise.all` block is **unchanged** on this branch (same on `main` at lines 64–67; only renumbered to 71–73 after edits above). **Do not use line 72** — it will not appear in the PR Files view.

Anchor on a **changed** line in the same file and point to the bulk mutation, e.g.:

| Anchor (in PR diff) | Why |
|---------------------|-----|
| **~line 111** — `useVideoLibraryColumns(..., sortBy, sortOrder)` | Added/changed in this PR |
| **~line 5–6** — import of `bulkUpdateVideoLibrary` | In first diff hunk (context) |

**Suggested comment text (paste on that anchor):**

> **High (pre-existing, still open on this branch):** In `bulkUpdateMutation`, status-only bulk still does `Promise.all(videoIds.map(id => patchVideoLibraryItem(id, { status })))` (~lines 71–73). Please use `bulkUpdateVideoLibrary({ videoIds, status })` like the branch below it — one request, up to 200 ids.

**Status:** Open (pre-existing; not introduced by server-side table work, but still blocks merge if you require fixing all Highs on touched pages)

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `listPartnersDirectory` vs paginated directory API | CRITICAL | ✅ Fixed | src/api/services/partnerService.ts | 27-31 |
| 2 | Partner edit/view uses paginated list only | HIGH | ✅ Fixed | src/pages/partners/onboarding/index.tsx | 40-48 |
| 3 | Reporting managers from paginated list | HIGH | ✅ Fixed | src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx | 106-109 |
| 4 | Role/type filter options from current page | MEDIUM | ✅ Fixed | src/pages/management/crew-users/dashboard/index.tsx | 59 |
| 5 | Video library bulk status uses N× PATCH instead of bulk API | HIGH | Open | src/pages/videoLibrary/dashboard/components/video-library-table.tsx | 71-73 (unchanged vs `main`; anchor PR comment ~111) |

## Positive notes

- App users and video library pages follow the established server-table pattern (`useServerTableControls`, meta query with `pageSize: 1`, `PaginationBar`, server sort via column `key` + `applyServerSort`).
- Video library detail uses `getVideoLibraryItem(id)` (not list page 1).
- Lookups for video library event/athlete filters come from `getVideoLibraryLookups`, not paginated rows.
- App-user bulk UI correctly calls a single `bulkPatchAppUsers` API (fix belongs on backend — see [backend.md](./backend.md) #3).
- Partner table sort keys aligned with narrowed backend partner sort map.

**Merge readiness:** Blocked on Open High #5 (frontend) and [backend.md](./backend.md) High #3 (app-user bulk patch). Do not archive to `Completed/` until both are Fixed or Accepted.
