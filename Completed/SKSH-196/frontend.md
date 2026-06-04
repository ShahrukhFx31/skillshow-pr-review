# Frontend PR Review — skillshow-admin-ui (`SKSH-196`)

**Repo:** skillshow-admin-ui  
**Base:** `main...HEAD`  
**Branch:** `SKSH-196`  
**Re-reviewed:** 2026-05-26 (final pass — all prior findings addressed)  
**Scope:** Import tool wizard (validate → import), API client flat responses, help page  
**Severity:** Critical, High, Medium  
**Findings:** 9 (0 Critical, 0 Open, 9 Fixed)  
**Review standard:** [frontend-system-prompt.md](../prompts/frontend-system-prompt.md) (Critical/High/Medium, Status column, styling exclusions)

### Excluded from this review

| Topic | Reason |
|--------|--------|
| Pagination / deferred validation | Per review request |
| Ad-hoc palette vs theme tokens | Per [frontend-system-prompt.md](../prompts/frontend-system-prompt.md) |
| Tailwind `!` on Ant overrides | Per frontend system prompt |

---

## Simulated import progress hides single blocking request

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/utils/import-job-polling.utils.ts`  
**Lines:** 35-132

**Re-verification:** `runWizardImportWithPolling` polls `getImportJobStatus`; progress uses `importProcessed` / `importTotal`. No fake interval progress.

---

## `ImportToolFooter` accepts a large optional prop surface

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/types/components.ts`  
**Lines:** 19-63

**Re-verification:** Discriminated `variant` union + step-specific footers under `import-tool-footer/`.

---

## Ternary-with-`null` for optional UI fragments

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/components/validate-import-step.tsx`  

**Re-verification:** Logical `&&` for optional alerts and dividers.

---

## Table cell renderers live under `utils/`

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/components/validation-table-cells.tsx`  

**Re-verification:** JSX render helpers moved to `components/`.

---

## Duplicate `onReUpload` and `onReUploadErrorRows` handlers

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx`  

**Re-verification:** Single `onReUploadCsv` for validate and complete footers.

---

## Wizard orchestration concentrated in page `index.tsx`

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/index.tsx`  

**Re-verification:** `useImportToolWizard` owns flow; page is layout-only.

---

## Entity type list duplicated across repos

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/types.ts`  

**Re-verification:** `ImportEntityContractChecks` compile-time contract over `IMPORT_ENTITY_TYPE_VALUES`.

---

## `stepContent` `useMemo` omits several handler dependencies

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx`  

**Re-verification:** Step rendering lives in wizard hook; no fragile page-level memo.

---

## Validation poll errors swallowed

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx`  
**Lines:** 88-125

**Also:** `src/pages/import-tool/dashboard/components/validate-import-step.tsx`  
**Lines:** 55-66

**Description (original):**  
Empty `catch` on background validation status polling left users stuck on “validation in progress”.

**Re-verification:**  
- `runValidationStatusPoll`: `toast.error(getImportValidationPollErrorMessage(error))`, `setValidationPollFailed(true)`, `finally` clears `isCheckingValidationStatus`  
- `onRetryValidationStatusCheck` bumps `validationPollAttempt` to re-run poll  
- `ValidateImportStep` shows warning alert with **Retry status check** button; hides in-progress alert when `validationPollFailed`  
- Poll failure state reset on re-upload, reset wizard, and successful poll retry (`setValidationPollFailed(false)`)

**PR comment:** Resolved — toast + failed state + retry action.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Fake import progress timer | HIGH | ✅ Fixed | `dashboard/utils/import-job-polling.utils.ts` | 35-132 |
| 2 | Large `ImportToolFooter` prop surface | MEDIUM | ✅ Fixed | `dashboard/types/components.ts` | 19-63 |
| 3 | Ternary-with-`null` JSX | MEDIUM | ✅ Fixed | `dashboard/components/validate-import-step.tsx` | 46-67 |
| 4 | JSX renderers in `utils/` | MEDIUM | ✅ Fixed | `dashboard/components/validation-table-cells.tsx` | — |
| 5 | Duplicate re-upload handlers | MEDIUM | ✅ Fixed | `dashboard/hooks/use-import-tool-wizard.tsx` | 200+ |
| 6 | Wizard logic in page `index.tsx` | MEDIUM | ✅ Fixed | `dashboard/index.tsx` | 13-49 |
| 7 | Cross-repo entity type duplication | MEDIUM | ✅ Fixed | `src/pages/import-tool/types.ts` | 14-32 |
| 8 | `stepContent` memo dependencies | MEDIUM | ✅ Fixed | `dashboard/hooks/use-import-tool-wizard.tsx` | — |
| 9 | Validation poll errors swallowed | MEDIUM | ✅ Fixed | `dashboard/hooks/use-import-tool-wizard.tsx` | 88-125 |

**New issues on re-review:** None (Critical / High / Medium).

**All tracked frontend issues are resolved on the current branch.**

**Positive notes:** Status polling for validation and import; 409 conflict resume; step footers; temp-password reset flow; `getImportValidationPollErrorMessage` distinguishes timeout vs generic failure.

**Skipped (per request):** Refreshing `validCount` / `invalidCount` from status API after deferred validation — **Accepted**, not a merge blocker.

### Supplemental prompt-only pass (import-tool PR)

| Check | Result |
|--------|--------|
| Poll cleanup on unmount | ✅ `validationPollCancelledRef` in `useEffect` return |
| Import poll cancellation | ✅ Progress driven from API; no leaked intervals |
| `cn()` for conditional classes | ✅ Used on dashboard card, footers, upload step (no conflicting utilities flagged) |
| `global.css` feature rules | ✅ None added |
| `@/ui` primitives | ⚠️ Ant Design + Typography used (consistent with other admin pages); not flagged per project pattern |
| Prop drilling | ✅ Addressed via footer variants + wizard hook |
| Out-of-scope palette / `!` | ✅ Not reported |

**Merge readiness:** No open Critical, High, or Medium blockers. All summary rows are ✅ Fixed.
