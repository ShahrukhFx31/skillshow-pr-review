# PR review (SKSH-381) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#236](https://github.com/SkillshowFx/skillshow/pull/236) |
| Branch | `SKSH-381` → `main` |
| Head | `9852b6d5cd7b6c0661815eca6cbca461c140e6ec` |
| Scope | Role-specific `order` override on role-permissions; login tree uses effective order |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `SECURITY-AUDIT-PRE-RELEASE.md` |
| Re-verify | 2026-07-13 — head `9852b6d5` (expand/dedupe/`buildOrderUpsertFields`) |

**Aligned with:** [frontend.md](./frontend.md)

### Protected modules

| Module | Status |
|--------|--------|
| list-query / aggregation / audit-log / change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| Route `authorize` / system-role guards | ✅ unchanged |
| `RolePermissionModel.collection.findOneAndUpdate` soft-delete bypass | Existing pattern; PR only adds `order` to `$set` when explicit |
| Joi on permission assignment body | ✅ `order` integer ≥ 0 or null |

### Positive notes

- Validation, types, response `roleOrder` / effective `order`, and login-response override + multi-role `Math.min` merge are coherent.
- `buildExpandedPermissionAssignments` + `dedupeExpandedPermissions` + `buildOrderUpsertFields` correctly omit `order` on expansion-only children and prefer explicit rows.
- Tests cover create/update persistence, login effective-order, parent/child race, and omit-`$set`-order for expanded-only children.

## GitHub comments

_No open findings._

## Findings

---
Expanded child rows force `order: null` and race with explicit overrides

Risk Level: HIGH
File Path: src/controllers/role.controller.ts
Lines: 388

Description:
**Contract** — Parent expansion pushed children **without** `order`, then create/update/assign always wrote `order: perm.order ?? null`. The Ant Design permission tree checks parents and children together, so the same `permissionId` often appeared twice in `expandedPermissions`: once from expansion (`order` undefined → `null`) and once from the explicit selection (may carry an override). `Promise.all` upserts both; last write wins. CRUD flags are identical on both rows so the race was mostly harmless before; **`order` is not**, so an explicit child override could be wiped to `null`.

Impact:
- Saving a role after setting a child menu order could silently drop that override when the parent is also checked.
- Non-deterministic which `order` persisted under concurrent upserts.

Recommendation:
Dedupe by `permissionId` before persist, preferring the explicit input row (with `order`) over expansion-only rows. Alternatively, omit `order` from `$set`/`create` when the assignment did not include the field.

**Re-verify (9852b6d5):** ✅ Fixed — `buildExpandedPermissionAssignments` omits `order` on children, `dedupeExpandedPermissions` prefers explicit rows, `buildOrderUpsertFields` only `$set`s `order` for explicit input (expanded-only uses `$setOnInsert: { order: null }`). Covered by new controller tests.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Expanded child rows force `order: null` and race with explicit overrides | HIGH | ✅ Fixed | `src/controllers/role.controller.ts` | 388 |

**Merge readiness:** No open Critical/High blockers on backend.
