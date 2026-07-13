# PR review (SKSH-381) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#336](https://github.com/SkillshowFx/skillshow-admin-ui/pull/336) |
| Branch | `SKSH-381` → `main` |
| Head | `e01353f11d9f74051b272d86eb60f71cabe55408` |
| Scope | Role form order overrides; search-safe permission tree check merge; shared `PermissionOrderField` |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, audit-log UI stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Positive notes

- `PermissionOrderField` is shared by card + detail editor (**DRY**).
- Search-filtered tree check merge (`mergeFilteredTreeCheckedKeys` + `visibleKeyIds` on shrink) correctly preserves hidden selections.
- Hydration uses `roleOrder`; submit sends `order`; selected list sorts by effective order — aligns with backend `attachRolePermissionCrud`.

## GitHub comments

### `src/pages/management/role/permission-order-field.tsx`
- **HIGH** — Order input allows non-integers rejected by API (line 34)
- **MEDIUM** — Helper copy hardcodes English instead of `translate` (line 42)

## Findings

---
Order input allows non-integers rejected by API

Risk Level: HIGH
File Path: src/pages/management/role/permission-order-field.tsx
Lines: 34

Description:
**Contract** — Backend Joi is `Joi.number().integer().min(0).allow(null)`. The field uses `type="number"` + `Number(value)`, so values like `1.5` pass `updateOrderOverride` (not NaN, ≥ 0) and are sent on save, causing a **400**. `step={1}` / `parseInt` / `Number.isInteger` gating is missing. Cross-check: `skillshow` `role.validation.ts` integer constraint.

Impact:
- Role create/update fails after entering a decimal order with a validation error instead of clamping/rejecting in the UI.
- Override map can hold floats that never round-trip cleanly.

Recommendation:
Parse as a non-negative integer before calling `onChange`, e.g. empty → `null`; otherwise `Number.parseInt(value, 10)` and ignore when `!Number.isInteger(n) || n < 0`. Mirror the same guard in `updateOrderOverride` (`!Number.isInteger(value)`). Optionally set `step={1}` on the input.
---

---
Helper copy hardcodes English instead of `translate`

Risk Level: MEDIUM
File Path: src/pages/management/role/permission-order-field.tsx
Lines: 40-43

Description:
**DRY** / i18n — Label uses `translate("sys.permission.order")`, but the helper paragraph is hardcoded English (`Leave empty to use permission default...`). Sibling role/permission UI strings go through `t(...)`.

Impact:
- Locale builds show mixed language on the order field.
- Copy cannot be updated via locale files.

Recommendation:
Add locale keys (e.g. `sys.permission.orderDefaultHint` / `sys.permission.orderDefaultHintWithValue`) and render via `translate`, interpolating `defaultOrder` when present.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Order input allows non-integers rejected by API | HIGH | Open | `src/pages/management/role/permission-order-field.tsx` | 34 |
| 2 | Helper copy hardcodes English instead of `translate` | MEDIUM | Open | `src/pages/management/role/permission-order-field.tsx` | 40-43 |

**Merge readiness:** Request changes — open High on order input vs Joi integer contract; one Medium i18n follow-up.
