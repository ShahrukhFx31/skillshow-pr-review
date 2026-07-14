# PR review (SKSH-395) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#238](https://github.com/SkillshowFx/skillshow/pull/238) |
| Branch | `SKSH-395` → `main` |
| Head | `cf6d3a14b366f8ecbbcdbf7025958e06b584b74c` |
| Scope | Admin edit-request `queue=paid_or_recent`, search `$or` nesting, payment/order/goals history; `source_changes_requested` status |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `SECURITY-AUDIT-PRE-RELEASE.md` |
| Re-verify | 2026-07-14 — head `cf6d3a14` (adds `source_changes_requested` to constants/types) |

**Aligned with:** [frontend.md](./frontend.md)

### Protected modules

| Module | Status |
|--------|--------|
| list-query / aggregation / audit-log / change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| Route `authorize` / RBAC | ✅ unchanged (admin edit-request routes) |
| List contract / validation | ✅ `queue` oxor’d with `paymentStatus`; status allow-list includes `source_changes_requested` |

### Positive notes

- `queue=paid_or_recent` is oxor’d with `paymentStatus`, constants-backed, covered by a service test.
- `applyAdminSearchFilter` nests existing `$or` under `$and` so search no longer drops the queue filter.
- Admin update history for payment / due date / order ID / goals only records when values change.
- `orderId` on admin list items is a coherent API improvement.
- New `source_changes_requested` status aligns admin list filter with stored source-file workflow (pairs with frontend #337).

## GitHub comments

_No open findings._

## Findings

_No Critical or High findings._

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | _(none)_ | — | — | — | — |

**Merge readiness:** No open Critical/High blockers on backend.
