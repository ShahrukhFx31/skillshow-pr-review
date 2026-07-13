# PR review (SKSH-381) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#336](https://github.com/SkillshowFx/skillshow-admin-ui/pull/336) |
| Branch | `SKSH-381` → `main` |
| Head | `8c46e278d88e835456b3700b933f4e608c1b143e` |
| Scope | Role form order overrides; search-safe permission tree check merge; shared `PermissionOrderField` |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-13 — head `8c46e278` (integer parse + i18n helpers) |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, audit-log UI stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Positive notes

- `PermissionOrderField` is shared by card + detail editor (**DRY**).
- Search-filtered tree check merge (`mergeFilteredTreeCheckedKeys` + `visibleKeyIds` on shrink) correctly preserves hidden selections.
- Hydration uses `roleOrder`; submit sends `order`; selected list sorts by effective order — aligns with backend `attachRolePermissionCrud`.
- `parseOrderInput` + `step={1}` + `Number.isInteger` in `updateOrderOverride` match Joi integer contract.
- Helper copy uses `sys.permission.orderHelper` / `orderHelperWithDefault` in `en_US/sys.json`.

## GitHub comments

_No open findings._

## Findings

---
Order input allows non-integers rejected by API

Risk Level: HIGH
File Path: src/pages/management/role/permission-order-field.tsx
Lines: 34

Description:
**Contract** — Backend Joi is `Joi.number().integer().min(0).allow(null)`. The field previously used `type="number"` + `Number(value)`, so values like `1.5` could pass into overrides and fail role save with **400**.

Impact:
- Role create/update failed after entering a decimal order.
- Override map could hold floats that never round-tripped cleanly.

Recommendation:
Parse as a non-negative integer before calling `onChange`; mirror `Number.isInteger` in `updateOrderOverride`; optionally `step={1}`.

**Re-verify (8c46e278):** ✅ Fixed — `parseOrderInput` requires `Number.isInteger` and `>= 0`; `step={1}`; `updateOrderOverride` rejects non-integers.
---

---
Helper copy hardcodes English instead of `translate`

Risk Level: MEDIUM
File Path: src/pages/management/role/permission-order-field.tsx
Lines: 40-43

Description:
**DRY** / i18n — Label used `translate("sys.permission.order")`, but the helper paragraph was hardcoded English.

Impact:
- Locale builds showed mixed language on the order field.
- Copy could not be updated via locale files.

Recommendation:
Add locale keys and render via `translate`, interpolating `defaultOrder` when present.

**Re-verify (8c46e278):** ✅ Fixed — uses `translate("sys.permission.orderHelperWithDefault", { defaultOrder })` / `orderHelper` with matching `en_US/sys.json` entries.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Order input allows non-integers rejected by API | HIGH | ✅ Fixed | `src/pages/management/role/permission-order-field.tsx` | 34 |
| 2 | Helper copy hardcodes English instead of `translate` | MEDIUM | ✅ Fixed | `src/pages/management/role/permission-order-field.tsx` | 40-43 |

**Merge readiness:** No open Critical/High/Medium blockers on frontend.
