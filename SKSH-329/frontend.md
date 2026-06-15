# Frontend PR Review — skillshow-admin-ui (`SKSH-329`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-329`  
**Base:** `main...HEAD` @ `ef39df5`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-12 (`ef39df5` — overlay failure copy restored, `EditRequestTableText` consolidation, typos/hidePagination fixed, `sorter: true` regression)  
**Scope:** Edit-request copy refresh (“internal revision” → “internal review”, Title Case labels), shared table text truncation, admin list column sort behavior (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 7 (0 Critical, 4 High, 3 Medium) — **1 High Open**, **2 High Fixed**, **2 High Accepted**, **3 Medium Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts` | **Not modified** ✅ |
| `destructive-action-confirm-modal.tsx` | **Not modified** — consumer refactor only ✅ |
| `antd.adapter.tsx`, audit-log components | Not modified |

### Files reviewed (26 changed)

| File | Change |
|------|--------|
| `src/components/edit-request/EditRequestTableText.tsx` | **New** — shared char-limited text + tooltip |
| `src/constants/edit-request-display.constants.ts` | `EDIT_REQUEST_TABLE_TEXT_DISPLAY_MAX_LENGTH` |
| `src/utils/edit-request-display.utils.ts` | `formatEditRequestTableTextPreview` |
| `src/constants/edit-request-activity-history.constants.ts` | Internal review label updates |
| `src/constants/edit-request-output-version-status.constants.ts` | Title Case status labels |
| `src/constants/edit-request-permission.constants.ts` | Title Case permission labels |
| `src/components/edit-request/assign-editor/constants/assign-editor.constants.ts` | Title Case assign-editor copy |
| `src/pages/adminEditRequest/components/admin-edit-request-columns.tsx` | `EditRequestTableText`; `sorter: true` (regression — #7) |
| `src/pages/adminEditRequest/hooks/use-admin-edit-request-insights-table-columns.tsx` | Same `sorter: true` pattern |
| `src/pages/adminEditRequest/**` | Copy, filters refactor, output actions |
| `src/pages/editRequest/**` | Copy, tables/cards, overlay, submit handling |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `EditRequestTableText` + `formatEditRequestTableTextPreview` — single component for title/note truncation | ✅ DRY (improved since initial review) |
| Activity-history labels aligned with backend default note string | ✅ Accepted (#2) — legacy DB notes may show old wording |
| Title Case applied across admin/athlete edit-request copy | ✅ Fixed (#6) — sidebar uses `ASSIGN_EDITOR_ASSIGNED_LABEL` |
| Athlete marketing “2 rounds” copy changed to “review” | ✅ Accepted (#3) — revert to “revisions” before merge |
| Submit failure overlay body message | ✅ Fixed (#1) |
| `hidePagination` contract on `AdminEditRequestTable` | ✅ Fixed (#5) |
| Admin list/insights `sorter: true` without sort handler | ❌ Regression — see #7 |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **DRY improvement:** Consolidated `EditRequestTableTitle` / `EditRequestTableNote` into one `EditRequestTableText` + single formatter.
- **Failure UX restored:** `RequestStatusOverlay` again shows `OVERLAY_FAILURE_MESSAGE` on failed submit.
- **Copy polish:** Double-space typos fixed; `hidePagination ? false : hiddenPagination` restored; `tableSkyLoading` retained on admin tables.
- **Backend alignment:** Manager history defaults fixed on backend ([#1](./backend.md)).

---

## GitHub comments

### 1. ~~`src/pages/editRequest/components/RequestStatusOverlay.tsx` line 23~~ — **Fixed**

Failure body message restored via `bodyMessage = isSuccess ? OVERLAY_SUCCESS_MESSAGE : OVERLAY_FAILURE_MESSAGE`.

### 2. ~~Legacy note normalization~~ — **Accepted**

### 3. ~~Athlete “2 rounds” copy incorrectly changed to “review”~~ — **Accepted**

### 4. ~~Double-space typos~~ — **Fixed**

### 5. ~~`hidePagination` contract~~ — **Fixed**

### 6. ~~`assignedEditorLabel` Title Case~~ — **Fixed** (sidebar copy removed; `ASSIGN_EDITOR_ASSIGNED_LABEL` used)

### 7. `src/pages/adminEditRequest/components/admin-edit-request-columns.tsx` line 70

**PR comment (line 70):** **High (Regression):** Client-side `sorter: (a, b) => …` compare functions were replaced with `sorter: true`, but `AdminEditRequestTable` has no `onChange` / `applyServerSort` wiring. Column headers show sort UI but rows do not reorder. Restore client compare functions or wire server sort end-to-end (same issue in insights columns).

---

## Findings

---
Submit failure overlay shows no error message

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx  
Lines: 17-27, 58-59

Description:
**UX regression (initial review).** Prior commit removed failure body text and `appToast.error` on submit failure.

Impact:
- Athletes saw “Request Failed” with no explanation.

Recommendation:
Restore `OVERLAY_FAILURE_MESSAGE` in overlay body for failure state.

**Re-review evidence:** `bodyMessage = isSuccess ? OVERLAY_SUCCESS_MESSAGE : OVERLAY_FAILURE_MESSAGE` and paragraph renders `{bodyMessage}`. Toast on error remains removed; overlay copy is sufficient per original either/or recommendation.

---

---
Legacy internal-revision default notes no longer normalized

Risk Level: HIGH  
**Status:** Accepted  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts  
Lines: 52-55

Description:
Historical `internal_revision_requested` rows may store `"Sent for internal revision"`.

**Accepted:** Legacy stored notes are out of scope for SKSH-329; acceptable to show historical copy verbatim.

---

---
Athlete “2 rounds” copy incorrectly changed to “review”

Risk Level: HIGH  
**Status:** Accepted  
File Path: skillshow-admin-ui/src/pages/editRequest/components/UploadVideosScreen.tsx  
Lines: 77

Description:
**Copy scope.** `UploadVideosScreen` and `EnhanceVideoScreen` use “Up to 2 rounds of review”. That describes the **athlete product offering** (revision rounds on delivered edits), not the crew/manager internal-review rename.

Impact:
- Confusing athlete-facing marketing copy.

Recommendation:
Revert both to `"Up to 2 rounds of revisions"`.

**Accepted:** Valid copy-scope finding. Revert athlete marketing copy to “revisions” in `UploadVideosScreen` and `EnhanceVideoScreen` before merge.

---

---
Double-space typos in internal review copy constants

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 99, 102

Description:
**Copy quality.** `reviewFailed` and `sectionTitle` had double spaces.

**Re-review evidence:** `reviewFailed: "Failed to submit internal review."`, `sectionTitle: "Internal review"`.

---

---
hidePagination no longer disables Table pagination

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx  
Lines: 122

Description:
**Contract.** `hidePagination` must force `pagination={false}` for server-sliced preview tables.

**Re-review evidence:** `pagination={hidePagination ? false : hiddenPagination}` restored.

---

---
Incomplete Title Case on assigned-editor sidebar label

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 262-274

Description:
**Global consistency.** `ADMIN_EDIT_REQUEST_SIDEBAR_CARD_COPY.assignedEditorLabel` was lowercase while assign-editor modal used Title Case.

**Re-review evidence:** `assignedEditorLabel` / `assignEditor` keys removed from sidebar copy; `AssignEditorField` uses `ASSIGN_EDITOR_ASSIGNED_LABEL` (“Assigned Editor:”).

---

---
Admin list column sort UI no longer sorts rows

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-columns.tsx  
Lines: 70-224

Description:
**Regression.** This PR replaces client-side compare functions (e.g. `sorter: (a, b) => a.title.localeCompare(b.title)`) with `sorter: true` on all sortable columns. `AdminEditRequestTable` and `AdminEditRequestInsightsTable` do not pass `onChange` with `applyServerSort`, and the list hooks do not expose `sortBy` / `sortOrder` to the API.

Impact:
- Column sort affordances appear but clicking headers does not reorder the current page (broken sort UX on My Edit Requests and Insights tabs).
- Regresses behavior that worked via in-page client sort before this PR.

Recommendation:
Either restore client-side compare functions:

```typescript
sorter: (a, b) => a.title.localeCompare(b.title),
```

Or wire full server sort: `useServerTableControls` + `applyServerSort` + API `sortBy`/`sortOrder` on list queries (match protected server-table contract).

Apply the same fix in `use-admin-edit-request-insights-table-columns.tsx`.

**PR comment (line 70):** **High (Regression):** `sorter: true` without Table `onChange` or compare functions breaks column sorting. Restore client `sorter` functions or wire server sort.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Submit failure overlay shows no error message | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx | 17-27, 58-59 | — |
| 2 | Legacy internal-revision default notes no longer normalized | HIGH | Accepted | skillshow-admin-ui/src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts | 52-55 | — |
| 3 | Athlete “2 rounds” copy incorrectly changed to “review” | HIGH | Accepted | skillshow-admin-ui/src/pages/editRequest/components/UploadVideosScreen.tsx | 77 | 77 |
| 4 | Double-space typos in internal review copy constants | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 99, 102 | — |
| 5 | hidePagination no longer disables Table pagination | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx | 122 | — |
| 6 | Incomplete Title Case on assigned-editor sidebar label | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 262-274 | — |
| 7 | Admin list column sort UI no longer sorts rows | HIGH | Open | skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-columns.tsx | 70-224 | 70 |

**Merge readiness:** **Not merge-ready** — 1 open High (#7 broken column sorting). Backend is merge-ready. Findings #2–#3 **Accepted**; #1, #4–#6 **Fixed** on branch. #3 accepted: revert athlete “2 rounds” copy to “revisions” before merge.
