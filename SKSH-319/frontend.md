# Frontend PR Review — skillshow-admin-ui (`SKSH-319`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-319`  
**Base:** `main...HEAD` @ `e853b0eb`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-12 (`e853b0eb` — app-user onboarding refactor bundled; upload-date export unchanged); **2026-06-12** (no new commits — finding #1 still open)  
**Scope:** Video Library import-tool integration + app-user sports-profile form refactor (Critical / High only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 1 (0 Critical, 1 High) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** ✅ |

### Files reviewed (13 changed)

**Video Library / import-tool (in scope)**

| File | Change |
|------|--------|
| `src/pages/import-tool/constants.ts` | `videoLibrary` entity type + labels |
| `src/pages/import-tool/dashboard/components/entity-selected-banner.tsx` | Configurable `hint`; layout tweak |
| `src/pages/import-tool/dashboard/components/select-entity-step.tsx` | Video-library workflow hint |
| `src/pages/import-tool/dashboard/components/upload-csv-step.tsx` | Entity-specific upload copy |
| `src/pages/import-tool/dashboard/constants/select-entity-step.constants.ts` | Sample row for `videoLibrary` |
| `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx` | Pass `entityType` to upload step |
| `src/pages/import-tool/dashboard/types/components.ts` | `hint`, `entityType` props |
| `src/pages/import-tool/dashboard/utils/import-sample.utils.ts` | `getVideoLibraryCsvHeaders` case |
| `src/pages/videoLibrary/dashboard/constants.ts` | Shared CSV labels, headers, hints, sample row |
| `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` | Use `VIDEO_LIBRARY_CSV_COLUMN_LABELS` |

**App-user onboarding (bundled — `e853b0eb`)**

| File | Change |
|------|--------|
| `src/pages/management/app-users/onboarding/components/app-user-form.tsx` | Pass `isAthlete: effectiveRole` to patch builder |
| `src/pages/management/app-users/onboarding/components/app-user-sports-profile-fields.tsx` | `SportSelectField` with `visible` prop |
| `src/pages/management/app-users/onboarding/utils.ts` | Safer patch payload trimming + `isAthlete` option |

### DRY / KISS / Global consistency scan

| Check | Verdict |
|-------|---------|
| `VIDEO_LIBRARY_CSV_COLUMN_LABELS` / `VIDEO_LIBRARY_CSV_HEADERS` single source for export + import template | ✅ DRY |
| Import-tool reuses video-library constants (hints, headers, sample) | ✅ |
| `ImportEntityContractChecks` satisfied | ✅ |
| CSV column order matches backend `IMPORT_TOOL_VIDEO_LIBRARY_HEADERS` | ✅ |
| Upload date export format vs backend read-only validation | ❌ Cross-stack — see #1 |
| App-user `buildAppUserPatchPayload(..., { isAthlete: effectiveRole })` | ✅ Fixes edit-mode role vs form role mismatch |
| App-user sport fields use `display:none` vs conditional unmount | ✅ Accepted — `handleSportChange` still clears invalid selects |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **DRY:** Shared CSV column labels prevent export/import header drift.
- **UX:** Workflow hints describe export → edit Event/Athlete → import clearly.
- **App-user fix:** Patch uses `effectiveRole` for sports-profile inclusion — avoids omitting profile on athlete edit when form role field differs.
- **Banner layout:** Entity-selected banner hint prop supports longer video-library copy without breaking download button alignment.

---

## GitHub comments (Open findings)

### 1. `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` line 22

**PR comment (line 22 — `formatDate(row.uploadedAt)` in `videoLibraryCsvRowEntries`):**

> **High (Global consistency / Regression):** This export formats `Upload date` with browser-local `formatDate(row.uploadedAt)`, but the backend import validator compares the CSV cell to server-local `formatTableExportDate(createdAt)` (`skillshow` PR — `src/utils/video-library-import.utils.ts` ~line 102). Unmodified exports can fail read-only validation across timezones. Either drop upload-date tamper checks on the backend (`1a9542a` approach) or use the same canonical date format (e.g. UTC) on both sides.

**GitHub anchor:** In the **skillshow-admin-ui** PR Files tab, open `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` and comment on the changed line containing `formatDate(row.uploadedAt)` (line **22** on the branch, not 20).

---

## Findings

---
Video Library CSV export date format breaks import read-only validation

Risk Level: HIGH  
**Status:** Open  
File Path: src/pages/videoLibrary/dashboard/export-video-library-csv.ts  
Lines: 22 (`formatDate(row.uploadedAt)` in `videoLibraryCsvRowEntries`)

Description:
**Global consistency / Cross-stack.** `exportVideoLibraryRowsAsCsv` (primary "Export Videos" action) formats `Upload date` with `formatDate(row.uploadedAt)` — browser-local `MM/DD/YYYY`.

Backend [finding #1](./backend.md) compares that cell to `formatTableExportDate(row.createdAt)` on the server. The branch previously fixed this on the backend by **omitting** upload-date validation (`1a9542a`); **`d96579e` reverted that fix**, so the frontend export format mismatch is blocking again.

Impact:
- False **"This column cannot be edited"** on `Upload date` for unmodified exports when browser TZ ≠ server TZ
- Core import workflow blocked

Recommendation:
**Short term (matches backend `1a9542a`):** No frontend change required if backend drops upload-date tamper check.

**If keeping upload-date validation:** format export with the same canonical helper as backend (e.g. UTC `YYYY-MM-DD` shared util).

**PR comment:** `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` line **22** — see GitHub comments above. Primary backend fix target: `skillshow` `src/utils/video-library-import.utils.ts` line **102**.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Video Library CSV export date format breaks import read-only validation | HIGH | Open | src/pages/videoLibrary/dashboard/export-video-library-csv.ts | 22 | 22 |

**Merge readiness:** **Not merge-ready** — 1 open High cross-stack blocker. Easiest fix: re-apply backend `1a9542a` (omit upload-date check). See [backend.md](./backend.md) #1.
