# PR review (SKSH-395) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#238](https://github.com/SkillshowFx/skillshow/pull/238) |
| Branch | `SKSH-395` → `main` |
| Head | `3180a12c9dd75bc904f142bf8daf656a18dc346b` |
| Scope | Admin edit-request `queue=paid_or_recent`, search `$or` nesting, payment/order/goals history |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `SECURITY-AUDIT-PRE-RELEASE.md` |

**Aligned with:** [frontend.md](./frontend.md)

### Protected modules

| Module | Status |
|--------|--------|
| list-query / aggregation / audit-log / change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| Route `authorize` / RBAC | ✅ unchanged (admin edit-request routes) |
| List contract / validation | ✅ `queue` oxor’d with `paymentStatus` |

### Positive notes

- `queue=paid_or_recent` is oxor’d with `paymentStatus`, constants-backed, covered by a service test.
- `applyAdminSearchFilter` nests existing `$or` under `$and` so search no longer drops the queue filter.
- Admin update history for payment / due date / order ID / goals only records when values change.
- `orderId` on admin list items is a coherent API improvement.

## GitHub comments

_No open findings._

## Findings

_No Critical or High findings._

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | _(none)_ | — | — | — | — |

**Merge readiness:** No open Critical/High blockers on backend.
