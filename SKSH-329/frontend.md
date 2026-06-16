# Frontend PR Review — skillshow-admin-ui (`SKSH-329`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-329`  
**Base:** `main...HEAD` @ `0627ab8`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-12 (`0627ab8` — server-sort wiring added; backend sort still missing; overlay/typos/hidePagination fixed)  
**Scope:** Edit-request copy refresh (“internal revision” → “internal review”, Title Case labels), shared table text truncation, admin list server-sort migration (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 7 (0 Critical, 4 High, 3 Medium) — **1 High Open**, **2 High Fixed**, **2 High Accepted**, **3 Medium Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts` | **Not modified** ✅ — consumed correctly |
| `destructive-action-confirm-modal.tsx` | **Not modified** ✅ |
| `antd.adapter.tsx`, audit-log components | Not modified |

### Files reviewed (37 changed)

| File | Change |
|------|--------|
| `src/components/edit-request/EditRequestTableText.tsx` | **New** — shared char-limited text + tooltip |
| `src/utils/edit-request-display.utils.ts` | `formatEditRequestTableTextPreview` |
| `src/pages/adminEditRequest/index.tsx` | `useServerTableControls`, sort in queryKey/API params |
| `src/pages/adminEditRequest/components/admin-edit-request-table.tsx` | `applyServerSort` onChange |
| `src/pages/adminEditRequest/components/admin-edit-request-insights-table.tsx` | `applyServerSort` onChange |
| `src/pages/adminEditRequest/components/admin-edit-request-columns.tsx` | `sorter: true`, `EditRequestTableText` |
| `src/pages/adminEditRequest/constants/admin-edit-request.constants.ts` | Copy + `*_SORT_KEYS` allow-lists |
| `src/api/services/adminEditRequestService.ts` | `sortBy` / `sortOrder` query params |
| `src/pages/editRequest/**` | Copy, tables/cards, overlay |
| `src/pages/dashboard/crew/.../edit-requests-columns.tsx` | `formatEditRequestDisplayId` DRY |
| `src/pages/management/crew-users/work/.../crew-user-work-columns.tsx` | Display-id util reuse |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `EditRequestTableText` + single preview formatter | ✅ DRY |
| `formatEditRequestDisplayId` reused outside admin list | ✅ |
| `useServerTableControls` + `applyServerSort` + `queryKey` deps | ✅ Contract wiring |
| Backend `adminListQuerySchema` accepts `sortBy`/`sortOrder` | ❌ Stripped — see #7 |
| Athlete marketing “2 rounds” copy | ✅ Accepted (#3) — intentional “review” wording |
| Submit failure overlay body message | ✅ Fixed (#1) |
| `hidePagination` contract | ✅ Fixed (#5) |
| Legacy note normalization | ✅ Accepted (#2) |

### Positive notes

- **Server-table contract:** Admin list + insights tables now use `useServerTableControls`, `applyServerSort`, hidden pagination, and sort params in `queryKey` — correct frontend pattern.
- **DRY:** `EditRequestTableText` consolidates title/note truncation; display-id formatting promoted to shared util.
- **Prior fixes retained:** Failure overlay copy, typos, `hidePagination`, assign-editor label via `ASSIGN_EDITOR_ASSIGNED_LABEL`.

---

## GitHub comments

### 1. ~~Submit failure overlay~~ — **Fixed**

### 2. ~~Legacy note normalization~~ — **Accepted**

### 3. ~~`src/pages/editRequest/components/UploadVideosScreen.tsx` line 77~~ — **Accepted**

“Up to 2 rounds of review” kept on athlete upload/enhance screens; accepted as intentional product copy for SKSH-329.

### 4. ~~Double-space typos~~ — **Fixed**

### 5. ~~`hidePagination` contract~~ — **Fixed**

### 6. ~~`assignedEditorLabel` Title Case~~ — **Fixed**

### 7. `src/pages/adminEditRequest/index.tsx` line 207

**PR comment (line 207):** **High (Contract / cross-stack):** `listParams` sends `sortBy`/`sortOrder`, but backend `adminListQuerySchema` and `adminEditRequestInsightsListQuerySchema` strip unknown query keys — column sort updates UI state but API order is unchanged. Add backend sort support (schema + service) or revert to client-side `sorter` compare functions until backend lands.

---

## Findings

---
Submit failure overlay shows no error message

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx  
Lines: 17-27, 58-59

**Re-review evidence:** `bodyMessage` uses `OVERLAY_FAILURE_MESSAGE` on failure; paragraph renders `{bodyMessage}`.

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

Description:
**Copy scope.** `UploadVideosScreen` and `EnhanceVideoScreen` use “Up to 2 rounds of review” instead of “revisions”.

**Accepted:** Intentional athlete-facing wording for SKSH-329; no revert to “revisions” required.

---

---
Double-space typos in internal review copy constants

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 99, 102

**Re-review evidence:** `reviewFailed: "Failed to submit internal review."`, `sectionTitle: "Internal review"`.

---

---
hidePagination no longer disables Table pagination

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx  
Lines: 131

**Re-review evidence:** `pagination={hidePagination ? false : hiddenPagination}` restored.

---

---
Incomplete Title Case on assigned-editor sidebar label

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 262-274

**Re-review evidence:** Sidebar copy no longer duplicates assign-editor labels; `AssignEditorField` uses `ASSIGN_EDITOR_ASSIGNED_LABEL`.

---

---
Admin list server sort params ignored by backend

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/index.tsx  
Lines: 204-213, 225-241

Description:
**Contract / cross-stack.** This PR migrates admin My Edit Requests + Insights tabs to server sort: `useServerTableControls`, `applyServerSort` on tables, `sortBy`/`sortOrder` in `queryKey`, and `listParams` / `insightsListParams` passed to `adminEditRequestService`.

Backend `adminListQuerySchema` and `adminEditRequestInsightsListQuerySchema` (`skillshow/src/validation/edit-request.validation.ts`) do **not** define `sortBy`/`sortOrder`. Validate middleware uses `stripUnknown: true`, so sort params are dropped before controllers run.

Impact:
- Column sort UI toggles and refetches, but list order from the API does not change.
- Worse than the prior client-sort regression: appears to work while data stays unsorted.

Recommendation:
Either (a) add backend sort validation + repository/service sort for admin list, assignment-returns, and feedback endpoints in the same ticket, or (b) revert column `sorter: true` + server-sort wiring and restore client-side compare functions until backend is ready.

**PR comment (line 207):** **High (Contract):** `sortBy`/`sortOrder` are sent but backend schemas strip them — sort UI is non-functional. Pair with backend schema/service work or revert to client sorters.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Submit failure overlay shows no error message | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx | 17-27, 58-59 | — |
| 2 | Legacy internal-revision default notes no longer normalized | HIGH | Accepted | skillshow-admin-ui/src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts | 52-55 | — |
| 3 | Athlete “2 rounds” copy incorrectly changed to “review” | HIGH | Accepted | skillshow-admin-ui/src/pages/editRequest/components/UploadVideosScreen.tsx | 77 | — |
| 4 | Double-space typos in internal review copy constants | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 99, 102 | — |
| 5 | hidePagination no longer disables Table pagination | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx | 131 | — |
| 6 | Incomplete Title Case on assigned-editor sidebar label | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 262-274 | — |
| 7 | Admin list server sort params ignored by backend | HIGH | Open | skillshow-admin-ui/src/pages/adminEditRequest/index.tsx | 204-241 | 207 |

**Merge readiness:** **Not merge-ready** — 1 open High (#7 non-functional server sort). Finding #3 **Accepted** (athlete “2 rounds of review” copy). Backend repo is merge-ready. For #7 either add backend sort support or revert to client-side sorters.
