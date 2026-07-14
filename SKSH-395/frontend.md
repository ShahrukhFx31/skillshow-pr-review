# PR review (SKSH-395) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#337](https://github.com/SkillshowFx/skillshow-admin-ui/pull/337) |
| Branch | `SKSH-395` → `main` |
| Head | `1488622081551f79e197b9cfc4fd4ad62a114002` |
| Scope | Admin EDR status/payment filters, Order ID column, Goals card split |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| pagination-bar, use-pagination, use-server-table-controls, table-sort, audit-log, DestructiveActionConfirmModal | **Not modified** ✅ |

### Positive notes

- Goals vs Payment/Order ID split; `AdminEditRequestOrderIdField` reused on list + detail.
- `toAdminEditRequestPaymentListQuery` maps payment vs `queue` cleanly.
- `statusFilter` / payment included in pagination `filterKey` and list `queryKey`.

## GitHub comments

### `src/pages/adminEditRequest/index.tsx`

- **L98** — Default payment filter is `paid`, not `paid_or_recent` (HIGH)
- **L113** — Status filter remaps `source_changes_requested` to `change_requested` (HIGH)

### `src/pages/adminEditRequest/viewRequest/index.tsx`

- **L367** — Shared `updateMutation.isPending` drives unrelated field loading (HIGH)

### `src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx`

- **L48** — Goals modal closes before save settles (MEDIUM)
- **L60** — Goals edit affordance uses ternary-null (MEDIUM)

### `src/pages/adminEditRequest/utils/list/admin-edit-request-export.utils.ts`

- **L66** — Payment export header not using shared column title (MEDIUM)

## Findings

---
Default payment filter is `paid`, not `paid_or_recent`

Risk Level: HIGH
File Path: src/pages/adminEditRequest/index.tsx
Lines: 98

Description:
**Contract** — Initial state is `ADMIN_EDIT_REQUEST_PAYMENT_STATUS.paid`, so every load sends `paymentStatus=paid`. Sibling backend (#238) treats `queue=paid_or_recent` as the admin queue (paid **or** created in last 7 days). Frontend exposes that queue as a select option but does not use it as default.

Impact:
- Unpaid EDRs created in the last week are hidden until the admin clears the filter or picks “Paid or last 7 days”.
- Diverges from the intended default admin queue.

Recommendation:
Default to `ADMIN_EDIT_REQUEST_LIST_QUEUE.paidOrRecent`. If “paid only” is intentional, document it and keep `paid_or_recent` as an explicit opt-in.
---

---
Status filter remaps `source_changes_requested` to `change_requested`

Risk Level: HIGH
File Path: src/pages/adminEditRequest/index.tsx
Lines: 113

Description:
**Contract** — For UI `changes_requested`, `mapUiStatusFilterToBackendStatus` yields `source_changes_requested`. This PR then forces `change_requested`, a different stored workflow (output/version changes), not source-file “changes requested.”

Impact:
- Filter labeled like source “Changes requested” returns version-`change_requested` rows.
- Real source-changes rows are missed.

Recommendation:
Remove the remapping. Send `mapUiStatusFilterToBackendStatus(statusFilter)` as-is. If admin Joi rejects `source_changes_requested`, fix allow-list with backend — do not substitute `change_requested`.
---

---
Shared `updateMutation.isPending` drives unrelated field loading

Risk Level: HIGH
File Path: src/pages/adminEditRequest/viewRequest/index.tsx
Lines: 367-384

Description:
`goalsSaving`, `orderIdLoading`, `paymentLoading`, and `expectedOutputCountSaving` all use `updateMutation.isPending`. Any PATCH lights loading on every sidebar control.

Impact:
- Editing Order ID disables payment and shows goals as saving.
- Overlapping edits mislead operators about what is in flight.

Recommendation:
Scope by mutation variables, e.g. `goalsSaving={updateMutation.isPending && updateMutation.variables?.goal !== undefined}` (same pattern as list `updatingOrderId`).
---

---
Goals modal closes before save settles

Risk Level: MEDIUM
File Path: src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx
Lines: 48

Description:
`handleSave` calls `onSave(draftGoals)` then immediately `setModalOpen(false)`. On failure the modal is already closed; `saving` barely shows.

Impact:
- Failed goals save looks like success until tags stay stale.

Recommendation:
Keep the modal open until mutation success (`onSuccess` / await settle), then close.
---

---
Payment export header not using shared column title

Risk Level: MEDIUM
File Path: src/pages/adminEditRequest/utils/list/admin-edit-request-export.utils.ts
Lines: 66

Description:
**DRY** — Outputs and Order ID CSV keys use `ADMIN_EDIT_REQUEST_COLUMN_TITLES`; Payment stays a literal `"Payment"`.

Impact:
- Export vs table title drift if copy changes.

Recommendation:
Use `ADMIN_EDIT_REQUEST_COLUMN_TITLES.payment` as the CSV key.
---

---
Goals edit affordance uses ternary-null

Risk Level: MEDIUM
File Path: src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx
Lines: 60

Description:
**JSX conditional rendering** — `extra={canEdit ? (<Button … />) : null}`.

Impact:
- Inconsistent with project preference for `condition && <Node />`.

Recommendation:
Prefer `extra={canEdit && (<Button … />)}`.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Default payment filter is `paid`, not `paid_or_recent` | HIGH | Open | `src/pages/adminEditRequest/index.tsx` | 98 |
| 2 | Status filter remaps `source_changes_requested` to `change_requested` | HIGH | Open | `src/pages/adminEditRequest/index.tsx` | 113 |
| 3 | Shared `updateMutation.isPending` drives unrelated field loading | HIGH | Open | `src/pages/adminEditRequest/viewRequest/index.tsx` | 367-384 |
| 4 | Goals modal closes before save settles | MEDIUM | Open | `src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx` | 48 |
| 5 | Payment export header not using shared column title | MEDIUM | Open | `src/pages/adminEditRequest/utils/list/admin-edit-request-export.utils.ts` | 66 |
| 6 | Goals edit affordance uses ternary-null | MEDIUM | Open | `src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx` | 60 |

**Merge readiness:** Not ready — 3 open High findings.
