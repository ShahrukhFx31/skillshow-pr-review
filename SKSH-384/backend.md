# PR review (SKSH-384) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#234](https://github.com/SkillshowFx/skillshow/pull/234) |
| Branch | `SKSH-384` → `main` |
| Head | `f8ef2268f082f7ce6c00ecd20ede518906843a39` |
| Scope | Default list page size 10→20; A–Z ordering for sports/roles/permissions/event types; partner multi-contact + host-code dashes; connections panel page sizes |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `SECURITY-AUDIT-PRE-RELEASE.md` |

**Aligned with:** [frontend.md](./frontend.md)

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Modified** — `DEFAULT_PAGE_SIZE` 10→20 (ticket scope; Accepted) |
| `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| No route auth / `authorize` weakened | ✅ |
| Partner contacts validated via Joi; create requires contacts or legacy name+email | ✅ |
| Host-code pattern widened to allow dashes only (no spaces/symbols) | ✅ |
| No IDOR / S3 / public-path changes | ✅ |

### Positive notes

- Admin list default page size aligned with frontend `DEFAULT_TABLE_PAGE_SIZE` (20).
- Connections panel `DEFAULT_LIMIT` / `PAGE_SIZE_OPTIONS` match admin-ui.
- Sport / event-type / permission-tree sort helpers are small, tested, and reused.
- Partner create custom Joi rule preserves legacy contact payloads.

## GitHub comments

### `src/services/partner-audit.service.ts`
- **HIGH** — Partner contacts audit omits phone (line 113)

## Findings

```markdown
---
Protected module changed (`list-query.validation.ts`)

Risk Level: CRITICAL
File Path: src/validation/list-query.validation.ts
Lines: 8-12

Description:
**Protected module** — PR changes frozen `LIST_QUERY_PAGINATION.DEFAULT_PAGE_SIZE` from 10 to 20. This is a global default for all `createListQuerySchema` consumers.

Impact:
- Every admin list that omits `pageSize` now returns 20 rows instead of 10.
- Shared module churn outside a dedicated list-query ticket is normally blocked.

Recommendation:
Accepted for SKSH-384: the ticket is explicitly about pagination defaults, and admin-ui mirrors `DEFAULT_TABLE_PAGE_SIZE = 20`. No further action required unless product wants the change reverted to a non-protected constant override per domain.
---
```

```markdown
---
Partner contacts audit omits phone

Risk Level: HIGH
File Path: src/services/partner-audit.service.ts
Lines: 107-120

Description:
**Contract** / audit completeness — `normalizeContactsAuditValue` serializes name, email, role, and department, but never `phone`. Primary phone may still surface via synced `contactNumber`, but phone-only edits on non-primary contacts (or any contact when `contactNumber` is unchanged) produce identical audit snapshots and are silently dropped.

Impact:
- Multi-contact phone updates are invisible in partner audit history.
- Incomplete audit trail for a field the create/edit UI collects.

Recommendation:
Include phone in the per-contact parts (after email), e.g. `if (contact.phone?.trim()) parts.push(contact.phone.trim())`, and add a unit test that a phone-only contacts change yields a non-empty audit diff.
---
```

PR comment (inline, 2–4 sentences):
`normalizeContactsAuditValue` drops `phone` from the contacts snapshot, so phone-only edits (especially on non-primary contacts) never appear in partner audit logs. Include trimmed phone in the serialized parts and cover a phone-only change in tests.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed (`list-query.validation.ts`) | CRITICAL | Accepted | `src/validation/list-query.validation.ts` | 8-12 |
| 2 | Partner contacts audit omits phone | HIGH | Open | `src/services/partner-audit.service.ts` | 107-120 |

**Merge readiness:** Request changes — 1 open High (contacts audit phone). Protected `DEFAULT_PAGE_SIZE` change Accepted for this ticket.
