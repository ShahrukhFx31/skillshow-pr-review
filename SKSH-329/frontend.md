# Frontend PR Review — skillshow-admin-ui (`SKSH-329`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-329`  
**Base:** `main...HEAD` @ `8bcd0596`  
**Initial review:** 2026-06-12  
**Scope:** Edit-request copy refresh (“internal revision” → “internal review”, Title Case labels), shared table title/note truncation components, admin table loading simplification, submit-failure overlay behavior (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 6 (0 Critical, 3 High, 3 Medium) — **2 High Open**, **1 High Accepted**, **3 Medium Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts` | **Not modified** ✅ |
| `destructive-action-confirm-modal.tsx` | **Not modified** — consumer refactor only (`EditDetailsSourceFilesSection`) ✅ |
| `antd.adapter.tsx`, audit-log components | Not modified |

### Files reviewed (29 changed)

| File | Change |
|------|--------|
| `src/components/edit-request/EditRequestTableTitle.tsx` | **New** — char-limited title + tooltip |
| `src/components/edit-request/EditRequestTableNote.tsx` | **New** — char-limited note + tooltip |
| `src/constants/edit-request-display.constants.ts` | **New** — 30-char display limits |
| `src/utils/edit-request-display.utils.ts` | **New** — shared preview formatters |
| `src/constants/edit-request-activity-history.constants.ts` | Internal review label updates |
| `src/constants/edit-request-output-version-status.constants.ts` | Title Case status labels |
| `src/constants/edit-request-permission.constants.ts` | Title Case permission labels |
| `src/components/edit-request/assign-editor/constants/assign-editor.constants.ts` | Title Case assign-editor copy |
| `src/pages/adminEditRequest/**` | Copy, columns, tables, insights, output actions |
| `src/pages/editRequest/**` | Copy, tables/cards, overlay, submit error handling |
| `src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts` | Default-note string update |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `EditRequestTableTitle` / `EditRequestTableNote` + `edit-request-display.utils` extracted for list truncation | ✅ DRY — adopted in admin list, insights, athlete list/cards |
| Activity-history labels aligned with backend default note string | ✅ Accepted (#2) — legacy DB notes may show old wording |
| Title Case applied across most admin/athlete edit-request copy | ⚠️ `assignedEditorLabel` still lowercase — see #6 |
| Athlete marketing “2 rounds” copy changed to “review” | ❌ Wrong scope — see #3 |
| `tableSkyLoading` removed from admin edit-request tables only | ✅ Scoped; other tables unchanged |
| Submit failure: overlay + toast both removed | ❌ UX regression — see #1 |
| `hidePagination` contract on `AdminEditRequestTable` | ❌ Regression risk — see #5 |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **Reusability:** `formatEditRequestTableTitlePreview` / `formatEditRequestTableNotePreview` centralize 30-char ellipsis + tooltip for edit-request list surfaces.
- **Global consistency:** `EditRequestTableTitle` wired through admin list, insights, athlete table, and mobile cards in one PR.
- **Destructive flow:** `EditDetailsSourceFilesSection` correctly hoists `removeConfirmItemName` for `DestructiveActionConfirmModal` without touching the frozen modal module.
- **Internal review rename:** Broad constant sweep updates UI labels, toasts, and activity-history display strings to “internal review”.

---

## GitHub comments

### 1. `src/pages/editRequest/components/RequestStatusOverlay.tsx` line 23

**PR comment (line 23):** **High (UX):** Submit failure now shows only the “Request Failed” title — the body message and `appToast.error` were both removed. Users get no explanation when `createEditRequest` fails. Restore a failure message in the overlay (or reinstate the toast) so the error state is actionable.

### 2. ~~`src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts` line 52~~ — **Accepted**

Legacy `"Sent for internal revision"` notes on pre-migration rows may display verbatim; no normalization helper required for this ticket.

### 3. `src/pages/editRequest/components/UploadVideosScreen.tsx` line 72

**PR comment (line 72):** **High (Copy scope):** “Up to 2 rounds of review” conflates athlete revision rounds with the internal-review workflow rename. This athlete-facing product copy should stay “revisions” (revert `UploadVideosScreen` and `EnhanceVideoScreen`).

### 4. `src/pages/adminEditRequest/constants/admin-edit-request.constants.ts` line 102

**PR comment (line 102):** **Medium:** `sectionTitle` and `reviewFailed` contain double spaces (`"Internal  review"`, `"internal  review"`). Trim to single spaces.

### 5. `src/pages/adminEditRequest/components/admin-edit-request-table.tsx` line 112

**PR comment (line 112):** **Medium (Contract):** `hidePagination` no longer forces `pagination={false}` on the desktop `Table`. Dashboard preview (`hidePagination`) relies on server-sliced rows; hidden pagination can client-slice an empty page when `page > 1`. Restore `hidePagination ? false : hiddenPagination`.

### 6. `src/pages/adminEditRequest/constants/admin-edit-request.constants.ts` line 263

**PR comment (line 263):** **Medium (Global consistency):** `ASSIGN_EDITOR_ASSIGNED_LABEL` was updated to “Assigned Editor:” but `ADMIN_EDIT_REQUEST_SIDEBAR_CARD_COPY.assignedEditorLabel` is still “Assigned editor:”. Align for Title Case consistency.

---

## Findings

---
Submit failure overlay shows no error message

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx  
Lines: 23-60

Description:
**UX regression.** This PR removes `OVERLAY_FAILURE_MESSAGE`, renders the success body only when `isSuccess`, and deletes `appToast.error(EDIT_REQUEST_SUBMIT_COPY.submitFailed)` from `editRequest/index.tsx` `onError`.

On `createEditRequest` failure the overlay opens with title “Request Failed”, failure icon, and the pro-editing link — but **no explanatory body text** and no toast.

Impact:
- Athletes see a failure modal with no guidance on what went wrong or what to do next.
- Support burden increases; users may assume the request was lost.

Recommendation:
Restore failure copy in the overlay:

```typescript
const OVERLAY_FAILURE_MESSAGE =
  "Something went wrong and your request could not be processed. Please try again or contact support.";
```

Render it when `isFailure`, or reinstate the toast in `onError` if the overlay body is intentionally minimal.

**PR comment (`RequestStatusOverlay.tsx` line 23):** **High (UX):** Submit failure now shows only the “Request Failed” title — the body message and `appToast.error` were both removed. Users get no explanation when `createEditRequest` fails. Restore a failure message in the overlay (or reinstate the toast).

---

---
Legacy internal-revision default notes no longer normalized

Risk Level: HIGH  
**Status:** Accepted  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts  
Lines: 52-55

Description:
**Global consistency / backward compatibility.** `resolveInternalRevisionRequestNoteForManager` gates on `"Sent for internal review"` only. Historical `internal_revision_requested` rows may still store `"Sent for internal revision"`.

Impact:
- Manager internal-review UI may show old wording on pre-migration requests only.
- New requests use the updated backend default string.

**Accepted:** Legacy stored notes are out of scope for SKSH-329; acceptable to show historical copy verbatim. New data uses “internal review” end-to-end.

---

---
Athlete “2 rounds” copy incorrectly changed to “review”

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/editRequest/components/UploadVideosScreen.tsx  
Lines: 72

Description:
**Copy scope.** `UploadVideosScreen` and `EnhanceVideoScreen` change “Up to 2 rounds of revisions” → “Up to 2 rounds of review”. That string describes the **athlete product offering** (revision rounds on delivered edits), not the crew/manager **internal review** workflow this ticket renames.

Impact:
- Confusing athlete-facing marketing copy; “review” implies a different workflow than paid revision rounds.
- Unrelated surface changed under an internal-review terminology ticket.

Recommendation:
Revert both files to “revisions”:

```typescript
"Up to 2 rounds of revisions"
```

**PR comment (line 72):** **High (Copy scope):** Revert athlete marketing copy to “revisions”; internal-review rename should not touch revision-round product language.

---

---
Double-space typos in internal review copy constants

Risk Level: MEDIUM  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 99, 102

Description:
**Copy quality.** `reviewFailed: "Failed to submit internal  review."` and `sectionTitle: "Internal  review"` contain double spaces before “review”.

Impact:
- Visible typos in manager internal-review section title and error toast.

Recommendation:
```typescript
reviewFailed: "Failed to submit internal review.",
sectionTitle: "Internal review",
```

**PR comment (line 102):** **Medium:** Fix double spaces in `sectionTitle` and `reviewFailed`.

---

---
hidePagination no longer disables Table pagination

Risk Level: MEDIUM  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx  
Lines: 112

Description:
**Contract.** Desktop table changed from `pagination={hidePagination ? false : hiddenPagination}` to always `pagination={hiddenPagination}`.

`AdminDashboardLiveOperationsSection` passes `hidePagination` with server-paginated preview rows (`page` always 1 today). Hidden pagination still configures Ant Table client-side slicing; if `page > 1` while `hidePagination` is true, the table can render an empty page even though `rows` contains data.

Impact:
- Latent regression for dashboard preview and any future `hidePagination` consumer.
- Violates the prop’s intent (show all passed rows, no table pagination).

Recommendation:
```typescript
pagination={hidePagination ? false : hiddenPagination}
```

**PR comment (line 112):** **Medium (Contract):** Restore `hidePagination ? false : hiddenPagination` for server-sliced preview tables.

---

---
Incomplete Title Case on assigned-editor sidebar label

Risk Level: MEDIUM  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts  
Lines: 263

Description:
**Global consistency.** Same PR updates `ASSIGN_EDITOR_ASSIGNED_LABEL` to “Assigned Editor:” but leaves `ADMIN_EDIT_REQUEST_SIDEBAR_CARD_COPY.assignedEditorLabel` as “Assigned editor:”.

Impact:
- Inconsistent casing between assign-editor modal and admin detail sidebar on the same screen.

Recommendation:
```typescript
assignedEditorLabel: "Assigned Editor:",
```

**PR comment (line 263):** **Medium:** Align `assignedEditorLabel` with “Assigned Editor:”.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Submit failure overlay shows no error message | HIGH | Open | skillshow-admin-ui/src/pages/editRequest/components/RequestStatusOverlay.tsx | 23-60 | 23 |
| 2 | Legacy internal-revision default notes no longer normalized | HIGH | Accepted | skillshow-admin-ui/src/pages/adminEditRequest/utils/internal-revision/admin-edit-request-internal-revision.utils.ts | 52-55 | — |
| 3 | Athlete “2 rounds” copy incorrectly changed to “review” | HIGH | Open | skillshow-admin-ui/src/pages/editRequest/components/UploadVideosScreen.tsx | 72 | 72 |
| 4 | Double-space typos in internal review copy constants | MEDIUM | Open | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 99, 102 | 102 |
| 5 | hidePagination no longer disables Table pagination | MEDIUM | Open | skillshow-admin-ui/src/pages/adminEditRequest/components/admin-edit-request-table.tsx | 112 | 112 |
| 6 | Incomplete Title Case on assigned-editor sidebar label | MEDIUM | Open | skillshow-admin-ui/src/pages/adminEditRequest/constants/admin-edit-request.constants.ts | 263 | 263 |

**Merge readiness:** **Not merge-ready** — 2 open High (submit failure UX, athlete revision-round copy) + 3 open Medium (typos, hidePagination contract, sidebar label). Finding #2 **Accepted** (legacy note wording). Fix remaining High blockers before merge.
