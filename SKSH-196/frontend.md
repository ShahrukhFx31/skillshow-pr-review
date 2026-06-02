# Frontend PR Review — skillshow-admin-ui (`SKSH-196`)

**Repo:** skillshow-admin-ui  
**Base:** `main...HEAD`  
**Branch:** `SKSH-196`  
**Re-reviewed:** 2026-05-26 (after developer fixes)  
**Scope:** Import tool wizard (validate → import), API client flat responses, help page  
**Severity:** Critical, High, Medium  
**Findings:** 9 (0 Critical, 1 Open, 7 Fixed, 1 new Medium)

### Excluded from this review

| Topic | Reason |
|--------|--------|
| Pagination / deferred validation | Per review request (`validationComplete` polling, rows beyond first batch, stale summary counts until re-validate) |
| Ad-hoc palette vs theme tokens | Per [frontend-system-prompt.md](../prompts/frontend-system-prompt.md) |
| Tailwind `!` on Ant overrides | Per frontend system prompt |

---

## Simulated import progress hides single blocking request

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/utils/import-job-polling.utils.ts`  
**Lines:** 35-132

**Description (original):**  
`runImportWithProgress` faked progress with a timer during one blocking `runImportJob`.

**Re-verification:**  
`runWizardImportWithPolling` handles **202** async acceptance, polls `getImportJobStatus`, and drives progress from `importProcessed` / `importTotal`. `import-progress-panel.tsx` uses `getImportProgressPercent` from polling utils. Timeout handling in `import-complete.utils.ts` (`isImportTimeoutError`).

**PR comment:** Resolved — real status polling replaces fake timer.

---

## `ImportToolFooter` accepts a large optional prop surface

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/types/components.ts`  
**Lines:** 19-63

**Also:** `src/pages/import-tool/dashboard/components/import-tool-footer/`

**Description (original):**  
Single footer with ~16 optional props for all steps.

**Re-verification:**  
Discriminated union (`variant: "selectEntity" | "uploadCsv" | "validate" | "complete" | "hidden"`) with step-specific prop types; step footers in `import-tool-footer/`; `useImportToolFooterProps` builds the variant.

**PR comment:** Resolved — step-specific footer variants.

---

## Ternary-with-`null` for optional UI fragments

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/components/validate-import-step.tsx`  
**Lines:** 37-44

**Also:** `src/pages/import-tool/dashboard/components/import-complete-step.tsx` (line 57)

**Description (original):**  
Optional UI used `condition ? <Node /> : null`.

**Re-verification:**  
Validate step uses `isBackgroundValidation && (<Alert … />)`. Complete step uses `hasFailures && <ValidationPartialImportHint … />`. Responsive table divider uses `columns.length > 1 && <div … />`.

**PR comment:** Resolved — logical AND for optional fragments.

---

## Table cell renderers live under `utils/`

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/components/validation-table-cells.tsx`  
**Lines:** 1-31

**Description (original):**  
JSX render helpers lived in `utils/validation-table-cell.tsx`.

**Re-verification:**  
Moved to `components/validation-table-cells.tsx`; imported by table hooks/components.

**PR comment:** Resolved — cells colocated under `components/`.

---

## Duplicate `onReUpload` and `onReUploadErrorRows` handlers

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx`  
**Lines:** 220+

**Description (original):**  
Two identical `useCallback`s for re-upload.

**Re-verification:**  
Single `onReUploadCsv` passed to validate and complete footers (`validate-step-footer.tsx`, `complete-step-footer.tsx`).

**PR comment:** Resolved — one `onReUploadCsv` handler.

---

## Wizard orchestration concentrated in page `index.tsx`

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/index.tsx`  
**Lines:** 13-49

**Also:** `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx`

**Description (original):**  
~288-line page held all wizard state and handlers.

**Re-verification:**  
`index.tsx` is layout-only; `useImportToolWizard` owns state, handlers, `stepContent`, and footer props.

**PR comment:** Resolved — `useImportToolWizard` extraction.

---

## Entity type list duplicated across repos

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/types.ts`  
**Lines:** 14-32

**Also:** `src/pages/import-tool/constants.ts` (`IMPORT_ENTITY_TYPE_VALUES`)

**Description (original):**  
No FE guard against backend enum drift.

**Re-verification:**  
`ImportEntityContractChecks` compile-time asserts labels, import targets, sample rows, and select options cover every `IMPORT_ENTITY_TYPE_VALUES` entry. Backend removed `user` entity; FE constants aligned (`skillshowUser` only).

**PR comment:** Resolved — compile-time contract (no runtime test, but type-level coverage).

---

## `stepContent` `useMemo` omits several handler dependencies

**Risk Level:** MEDIUM  
**Status:** ✅ Fixed  
**File Path:** `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx`

**Description (original):**  
Large `stepContent` memo in page `index.tsx` with narrow deps.

**Re-verification:**  
Step rendering moved into `useImportToolWizard`; page no longer memoizes `stepContent` directly.

**PR comment:** Resolved — logic moved to wizard hook.

---

## Validation status poll errors swallowed

**Risk Level:** MEDIUM  
**Status:** Open  
**File Path:** `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx`  
**Lines:** 111-122

**Description:**  
While on the validate step, background validation completion is polled via `pollUntilValidationComplete`. Failures in `getImportJobStatus` throw out of the poll loop, but the outer `catch` is empty with comment “non-fatal”. The user keeps seeing “validation in progress” with no toast and import stays disabled.

**Impact:**

- Transient network/API errors leave the wizard stuck until re-upload
- Hard to distinguish slow validation from a failed poll

**Recommendation:**  
On catch, `toast.error` with retry guidance and set a `validationPollFailed` flag to stop the in-progress alert; optionally offer “Retry status check”.

```ts
} catch (error) {
  toast.error(getImportValidationPollErrorMessage(error));
  setValidationPollFailed(true);
}
```

**PR comment (line 120):**  
**Medium:** Validation poll errors are swallowed — user can stay on “validation in progress” forever. Please toast and surface a retry when status polling fails.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Fake import progress timer | HIGH | ✅ Fixed | `dashboard/utils/import-job-polling.utils.ts` | 35-132 |
| 2 | Large `ImportToolFooter` prop surface | MEDIUM | ✅ Fixed | `dashboard/types/components.ts` | 19-63 |
| 3 | Ternary-with-`null` JSX | MEDIUM | ✅ Fixed | `dashboard/components/validate-import-step.tsx` | 37-44 |
| 4 | JSX renderers in `utils/` | MEDIUM | ✅ Fixed | `dashboard/components/validation-table-cells.tsx` | 1-31 |
| 5 | Duplicate re-upload handlers | MEDIUM | ✅ Fixed | `dashboard/hooks/use-import-tool-wizard.tsx` | 220+ |
| 6 | Wizard logic in page `index.tsx` | MEDIUM | ✅ Fixed | `dashboard/index.tsx` | 13-49 |
| 7 | Cross-repo entity type duplication | MEDIUM | ✅ Fixed | `src/pages/import-tool/types.ts` | 14-32 |
| 8 | `stepContent` memo dependencies | MEDIUM | ✅ Fixed | `dashboard/hooks/use-import-tool-wizard.tsx` | — |
| 9 | Validation poll errors swallowed | MEDIUM | Open | `dashboard/hooks/use-import-tool-wizard.tsx` | 111-122 |

**New issues on re-review:** 1 Medium (#9). No new Critical/High.

**Positive notes:** `getImportJobStatus` + validation/import polling; `ImportJobConflictError` for 409 resume; step footers; `runWizardImportWithPolling`; password-reset flow for temp-password users; feature layout unchanged and solid.

**Skipped (per request):** After validation poll completes, `validCount` / `invalidCount` are not refreshed from API (status response has no counts; only `validationComplete` flipped) — acceptable under pagination exclusion.
