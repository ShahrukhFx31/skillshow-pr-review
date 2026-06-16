# Frontend PR Review — skillshow-admin-ui (`SKSH-329`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-329`  
**Base:** `main...HEAD` @ `558a495`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-12 (`558a495` — server-sort reverted; client `sorter` compare functions restored)  
**Scope:** Edit-request copy refresh (“internal revision” → “internal review”, Title Case labels), shared table text truncation, admin list column sort (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 7 (0 Critical, 4 High, 3 Medium) — **0 Open**, **3 High Fixed**, **2 High Accepted**, **3 Medium Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts` | **Not modified** ✅ |
| `destructive-action-confirm-modal.tsx` | **Not modified** ✅ |
| `antd.adapter.tsx`, audit-log components | Not modified |

### Files reviewed (35 changed)

| File | Change |
|------|--------|
| `src/components/edit-request/EditRequestTableText.tsx` | **New** — shared char-limited text + tooltip |
| `src/utils/edit-request-display.utils.ts` | `formatEditRequestTableTextPreview` |
| `src/pages/adminEditRequest/components/admin-edit-request-columns.tsx` | `EditRequestTableText`, client `sorter` compare fns |
| `src/pages/adminEditRequest/hooks/use-admin-edit-request-insights-table-columns.tsx` | Client `sorter` compare fns |
| `src/pages/adminEditRequest/index.tsx` | `usePagination` (no server-sort params) |
| `src/pages/adminEditRequest/constants/admin-edit-request.constants.ts` | Copy refresh |
| `src/pages/editRequest/**` | Copy, tables/cards, overlay |
| `src/pages/dashboard/crew/.../edit-requests-columns.tsx` | `formatEditRequestDisplayId` DRY |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `EditRequestTableText` + single preview formatter | ✅ DRY |
| `formatEditRequestDisplayId` reused outside admin list | ✅ |
| Admin list uses client-side `sorter` compare (no orphan `sorter: true`) | ✅ Fixed (#7) |
| Athlete marketing “2 rounds” copy | ✅ Accepted (#3) |
| Submit failure overlay body message | ✅ Fixed (#1) |
| `hidePagination` contract | ✅ Fixed (#5) |
| Legacy note normalization | ✅ Accepted (#2) |
| Protected modules untouched | ✅ |

### Positive notes

- **Sort fix:** Reverted interim server-sort wiring; restored client-side compare functions on admin list + insights columns — sorting works on current page without backend schema changes.
- **DRY:** `EditRequestTableText` consolidates title/note truncation; display-id formatting promoted to shared util.
- **Copy sweep:** Internal review terminology and Title Case applied across admin/athlete edit-request surfaces.

---

## GitHub comments

_No open findings — prior comments resolved or accepted on branch._

---

## Findings

---
Submit failure overlay shows no error message

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx  
Lines: 17-27, 58-59

**Re-review evidence:** `bodyMessage` uses `OVERLAY_FAILURE_MESSAGE` on failure.

---

---
Legacy internal-revision default notes no longer normalized

Risk Level: HIGH  
**Status:** Accepted  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts  
Lines: 52-55

**Accepted:** Legacy stored notes may show old wording verbatim; out of scope for SKSH-329.

---

---
Athlete “2 rounds” copy incorrectly changed to “review”

Risk Level: HIGH  
**Status:** Accepted  
File Path: skillshow-admin-ui/src/pages/editRequest/components/UploadVideosScreen.tsx  
Lines: 77

**Accepted:** Intentional athlete-facing wording for SKSH-329.

---

---
Double-space typos in internal review copy constants

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 99, 102

---

---
hidePagination no longer disables Table pagination

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx  
Lines: 122

---

---
Incomplete Title Case on assigned-editor sidebar label

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 262-274

---

---
Admin list server sort params ignored by backend

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-columns.tsx  
Lines: 71-231

Description:
**Contract / cross-stack.** Prior commit wired `useServerTableControls` + API `sortBy`/`sortOrder` while backend schemas stripped those params.

Recommendation:
Revert to client-side `sorter` compare functions until backend sort lands.

**Re-review evidence (`558a495`):** Server-sort wiring removed from `index.tsx` and `adminEditRequestService`; `usePagination` restored; columns use client compare fns (e.g. `sorter: (a, b) => a.title.localeCompare(b.title)`); insights columns same pattern; `enableSorting: false` strips sorters for dashboard preview.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Submit failure overlay shows no error message | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx | 17-27, 58-59 | — |
| 2 | Legacy internal-revision default notes no longer normalized | HIGH | Accepted | skillshow-admin-ui/src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts | 52-55 | — |
| 3 | Athlete “2 rounds” copy incorrectly changed to “review” | HIGH | Accepted | skillshow-admin-ui/src/pages/editRequest/components/UploadVideosScreen.tsx | 77 | — |
| 4 | Double-space typos in internal review copy constants | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 99, 102 | — |
| 5 | hidePagination no longer disables Table pagination | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx | 122 | — |
| 6 | Incomplete Title Case on assigned-editor sidebar label | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 262-274 | — |
| 7 | Admin list server sort params ignored by backend | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-columns.tsx | 71-231 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High/Medium blockers. All findings Fixed or Accepted.
