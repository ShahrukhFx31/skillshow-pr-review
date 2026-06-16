# Frontend PR Review — skillshow-admin-ui (`SKSH-319`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-319`  
**Base:** `main...HEAD` @ `e853b0eb`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-16 (`e853b0eb` unchanged; backend `2666c77` changed contract behavior)  
**Scope:** Video Library import-tool integration + app-user sports-profile form refactor (Critical / High only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 0 (0 Critical, 0 High) — **0 Open**

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
| Upload date export format vs backend validation | ⚠️ Depends on backend contract; frontend path unchanged |
| App-user `buildAppUserPatchPayload(..., { isAthlete: effectiveRole })` | ✅ Fixes edit-mode role vs form role mismatch |
| App-user sport fields use `display:none` vs conditional unmount | ✅ Accepted — `handleSportChange` still clears invalid selects |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **DRY:** Shared CSV column labels prevent export/import header drift.
- **UX:** Workflow hints describe export → edit Event/Athlete → import clearly.
- **App-user fix:** Patch uses `effectiveRole` for sports-profile inclusion — avoids omitting profile on athlete edit when form role field differs.
- **Banner layout:** Entity-selected banner hint prop supports longer video-library copy without breaking download button alignment.

### Developer response (upload-date validation)

Team requirement remains: Upload date is non-editable and edited values should produce a validation error. Backend commit `2666c77` changed this behavior by removing Upload-date validation; blocker is tracked in [backend.md](./backend.md).

---

## GitHub comments (Open findings)

No frontend-only High/Critical findings.

---

## Findings

No open High/Critical findings in this frontend diff.

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| — | — | — | — | — | — | — |

**Merge readiness:** **Merge-ready** — no open Critical/High blockers across frontend/backend reports (backend High is accepted).
