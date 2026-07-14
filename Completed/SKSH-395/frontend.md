# PR review (SKSH-395) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#337](https://github.com/SkillshowFx/skillshow-admin-ui/pull/337) |
| Branch | `SKSH-395` → `main` |
| Head | `07812c2d3404195a956be9fa0df827027d187de7` |
| Scope | Admin EDR status/payment filters, Order ID column, Goals card split |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-14 — head `07812c2d` (async goals, scoped pending, default queue, status map, export title) |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| pagination-bar, use-pagination, use-server-table-controls, table-sort, audit-log, DestructiveActionConfirmModal | **Not modified** ✅ |

### Positive notes

- Goals vs Payment/Order ID split; `AdminEditRequestOrderIdField` reused on list + detail.
- `toAdminEditRequestPaymentListQuery` maps payment vs `queue` cleanly.
- `statusFilter` / payment included in pagination `filterKey` and list `queryKey`.
- Goals modal awaits `onSave` and stays open on failure; sidebar loading scoped by mutation body fields.

## GitHub comments

_No open findings._

## Findings

---
Default payment filter is `paid`, not `paid_or_recent`

Risk Level: HIGH
File Path: src/pages/adminEditRequest/index.tsx
Lines: 98

Description:
**Contract** — Initial state was `ADMIN_EDIT_REQUEST_PAYMENT_STATUS.paid`.

**Re-verify (07812c2d):** ✅ Fixed — defaults to `ADMIN_EDIT_REQUEST_LIST_QUEUE.paidOrRecent`.
---

---
Status filter remaps `source_changes_requested` to `change_requested`

Risk Level: HIGH
File Path: src/pages/adminEditRequest/index.tsx
Lines: 113

Description:
**Contract** — Remap forced wrong stored status.

**Re-verify (07812c2d):** ✅ Fixed — uses `mapUiStatusFilterToBackendStatus(statusFilter)` as-is (aligned with backend allow-list for `source_changes_requested`).
---

---
Shared `updateMutation.isPending` drives unrelated field loading

Risk Level: HIGH
File Path: src/pages/adminEditRequest/viewRequest/index.tsx
Lines: 367-384

Description:
All sidebar fields shared one pending flag.

**Re-verify (07812c2d):** ✅ Fixed — `goalsSaving` / `orderIdLoading` / `paymentLoading` / `expectedOutputCountSaving` scoped via `updateBody` fields.
---

---
Goals modal closes before save settles

Risk Level: MEDIUM
File Path: src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx
Lines: 48

Description:
Modal closed immediately after firing save.

**Re-verify (07812c2d):** ✅ Fixed — `await onSave(...)` then close; catch keeps modal open on failure.
---

---
Payment export header not using shared column title

Risk Level: MEDIUM
File Path: src/pages/adminEditRequest/utils/list/admin-edit-request-export.utils.ts
Lines: 66

Description:
Literal `"Payment"` CSV key.

**Re-verify (07812c2d):** ✅ Fixed — `[ADMIN_EDIT_REQUEST_COLUMN_TITLES.payment]`.
---

---
Goals edit affordance uses ternary-null

Risk Level: MEDIUM
File Path: src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx
Lines: 60

Description:
`canEdit ? … : null`.

**Re-verify (07812c2d):** ✅ Fixed — `canEdit && (…)` .
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Default payment filter is `paid`, not `paid_or_recent` | HIGH | ✅ Fixed | `src/pages/adminEditRequest/index.tsx` | 98 |
| 2 | Status filter remaps `source_changes_requested` to `change_requested` | HIGH | ✅ Fixed | `src/pages/adminEditRequest/index.tsx` | 113 |
| 3 | Shared `updateMutation.isPending` drives unrelated field loading | HIGH | ✅ Fixed | `src/pages/adminEditRequest/viewRequest/index.tsx` | 367-384 |
| 4 | Goals modal closes before save settles | MEDIUM | ✅ Fixed | `src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx` | 48 |
| 5 | Payment export header not using shared column title | MEDIUM | ✅ Fixed | `src/pages/adminEditRequest/utils/list/admin-edit-request-export.utils.ts` | 66 |
| 6 | Goals edit affordance uses ternary-null | MEDIUM | ✅ Fixed | `src/pages/adminEditRequest/components/admin-edit-request-goals-card.tsx` | 60 |

**Merge readiness:** No open Critical/High/Medium blockers on frontend.
