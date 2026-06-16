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
| Upload date export format vs backend read-only validation | ❌ TZ mismatch — canonical format needed (#1); omit check rejected for UX |
| App-user `buildAppUserPatchPayload(..., { isAthlete: effectiveRole })` | ✅ Fixes edit-mode role vs form role mismatch |
| App-user sport fields use `display:none` vs conditional unmount | ✅ Accepted — `handleSportChange` still clears invalid selects |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **DRY:** Shared CSV column labels prevent export/import header drift.
- **UX:** Workflow hints describe export → edit Event/Athlete → import clearly.
- **App-user fix:** Patch uses `effectiveRole` for sports-profile inclusion — avoids omitting profile on athlete edit when form role field differs.
- **Banner layout:** Entity-selected banner hint prop supports longer video-library copy without breaking download button alignment.

### Developer response (upload-date validation)

Team confirmed **`1a9542a` (omit upload-date check) is not acceptable**: Upload date is intentionally non-editable — if validation is skipped, users who change the cell see no error and assume the edit applied, which is more confusing than a false TZ failure. **`d96579e` correctly restored read-only validation.** Remaining work: align **canonical date formatting** on export and validation so unmodified exports pass while deliberate edits still surface **"This column cannot be edited"**.

---

## GitHub comments (Open findings)

### 1. `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` line 22

**PR comment (line 22 — `formatDate(row.uploadedAt)` in `videoLibraryCsvRowEntries`):**

> **High (Global consistency):** Keep upload-date read-only validation (correct UX when users edit the cell). Fix the TZ false-positive by using the **same canonical date helper** on export and in `video-library-import.utils.ts` (e.g. UTC `MM/DD/YYYY` or shared `formatVideoLibraryImportUploadDate`). Do **not** drop the check — omitting it (`1a9542a`) hides errors when users edit Upload date even though import never persists that field.

**GitHub anchor:** In the **skillshow-admin-ui** PR Files tab, open `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` and comment on the changed line containing `formatDate(row.uploadedAt)` (line **22** on the branch, not 20).

---

## Findings

---
Upload date export uses browser TZ; validation uses server TZ (read-only check must stay)

Risk Level: HIGH  
**Status:** Open  
File Path: src/pages/videoLibrary/dashboard/export-video-library-csv.ts  
Lines: 22 (`formatDate(row.uploadedAt)` in `videoLibraryCsvRowEntries`)

Description:
**Global consistency / Cross-stack.** `exportVideoLibraryRowsAsCsv` formats `Upload date` with browser-local `formatDate(row.uploadedAt)` (`MM/DD/YYYY`). Backend [finding #1](./backend.md) compares that cell to server-local `formatTableExportDate(row.createdAt)` in `validateReadOnlyImportCells`. When browser TZ ≠ server TZ, **unmodified** exports can falsely fail with **"This column cannot be edited"**.

**Developer response (accepted):** Commit `1a9542a` omitted upload-date validation to avoid TZ false positives, but that was **reverted in `d96579e`** because it caused worse UX: users who edit Upload date saw **no validation error** even though import never updates that field — implying the edit worked. Product intent: Upload date stays **read-only**; validation **must** error when the cell is changed. Omitting the check is not an option.

Impact:
- False read-only errors on unmodified exports (TZ mismatch) block the export → import workflow
- Dropping validation (per `1a9542a`) would confuse users who edit Upload date

Recommendation:
**Required fix (both repos):** Introduce one canonical formatter (e.g. UTC `dayjs.utc(iso).format("MM/DD/YYYY")` or shared constant) used in:
1. `export-video-library-csv.ts` line 22 (replace `formatDate`)
2. `mapImportRowsToValidationData` / `validateReadOnlyImportCells` in `video-library-import.utils.ts`

Add a cross-TZ unit test (fixed ISO instant where local formatting diverges). Unmodified exports pass; edited Upload date still fails with `VIDEO_LIBRARY_IMPORT_READ_ONLY_COLUMN_MESSAGE`.

**PR comment:** `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` line **22** — see GitHub comments above. Pair with `skillshow` `src/utils/video-library-import.utils.ts` line **102**.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Upload date export uses browser TZ; validation uses server TZ (read-only check must stay) | HIGH | Open | src/pages/videoLibrary/dashboard/export-video-library-csv.ts | 22 | 22 |

**Merge readiness:** **Not merge-ready** — 1 open High cross-stack blocker. Fix: shared canonical upload-date formatting on export + validation (not omit check). See [backend.md](./backend.md) #1.
