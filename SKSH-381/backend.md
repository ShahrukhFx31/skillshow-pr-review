# PR review (SKSH-381) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#236](https://github.com/SkillshowFx/skillshow/pull/236) |
| Branch | `SKSH-381` → `main` |
| Head | `86ee231350950e33046ab1b3ff49a7c0fa5c9ca0` |
| Scope | Role-specific `order` override on role-permissions; login tree uses effective order |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `SECURITY-AUDIT-PRE-RELEASE.md` |

**Aligned with:** [frontend.md](./frontend.md)

### Protected modules

| Module | Status |
|--------|--------|
| list-query / aggregation / audit-log / change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| Route `authorize` / system-role guards | ✅ unchanged |
| `RolePermissionModel.collection.findOneAndUpdate` soft-delete bypass | Existing pattern; PR only adds `order` to `$set` |
| Joi on permission assignment body | ✅ `order` integer ≥ 0 or null |

### Positive notes

- Validation, types, response `roleOrder` / effective `order`, and login-response override + multi-role `Math.min` merge are coherent.
- Tests cover create/update persistence and login effective-order behavior.

## GitHub comments

### `src/controllers/role.controller.ts`
- **HIGH** — Expanded child rows force `order: null` and race with explicit overrides (line 388)

## Findings

---
Expanded child rows force `order: null` and race with explicit overrides

Risk Level: HIGH
File Path: src/controllers/role.controller.ts
Lines: 388

Description:
**Contract** — Parent expansion pushes children **without** `order`, then create/update/assign always write `order: perm.order ?? null`. The Ant Design permission tree checks parents and children together, so the same `permissionId` often appears twice in `expandedPermissions`: once from expansion (`order` undefined → `null`) and once from the explicit selection (may carry an override). `Promise.all` upserts both; last write wins. CRUD flags are identical on both rows so the race was mostly harmless before; **`order` is not**, so an explicit child override can be wiped to `null`. Same pattern at create (~214) and assignPermissions (~624).

Impact:
- Saving a role after setting a child menu order can silently drop that override when the parent is also checked.
- Non-deterministic which `order` persists under concurrent upserts.

Recommendation:
Dedupe by `permissionId` before persist, preferring the explicit input row (with `order`) over expansion-only rows. Alternatively, omit `order` from `$set`/`create` when the assignment did not include the field (`'order' in perm`), so expansion-only children leave existing overrides alone:

```typescript
...(Object.prototype.hasOwnProperty.call(perm, "order")
  ? { order: perm.order ?? null }
  : {}),
```

Apply the same fix in `createRole`, `updateRole`, and `assignPermissions`.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Expanded child rows force `order: null` and race with explicit overrides | HIGH | Open | `src/controllers/role.controller.ts` | 388 |

**Merge readiness:** Request changes — open High on child expansion wiping role-specific `order`.
