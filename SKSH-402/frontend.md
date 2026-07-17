# PR review (SKSH-402) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#348](https://github.com/SkillshowFx/skillshow-admin-ui/pull/348) |
| Branch | `SKSH-402` → `main` |
| Head | `3b5ed29ceca2dd03b0114826a750b216727d9dd5` |
| Scope | Crew list columns/sort; SkillShow Users path → `/settings/...`; crew onboarding skip/branding; audit label helper |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

### Protected modules

| Module | Status |
|--------|--------|
| `src/utils/audit-log.utils.ts` | **Modified** ❌ — see Critical finding |
| Pagination / destructive-confirm | **Not modified** ✅ |

### Positive notes

- Crew list columns/sort keys align with backend #243.
- Legacy redirect for SkillShow Users mirrors the partners pattern (`legacy-redirect.tsx` + path mapper).
- Skip onboarding moved into `StepShell` (available on every step) looks intentional.

## GitHub comments

### `src/utils/audit-log.utils.ts`
- **L5** — Protected module changed (CRITICAL)

### `src/pages/management/skillshow-users/breadcrumb.ts`
- **L5** — Path migration may 404 without permission route update (HIGH)

## Findings

---
Protected module changed: `audit-log.utils.ts`

Risk Level: CRITICAL
File Path: src/utils/audit-log.utils.ts
Lines: 5-21

Description:
**Protected module** — PR extends `buildAuditFieldLabelMap` for `leftFields`/`rightFields` two-column form sections. SKSH-402 does not explicitly authorize edits to this frozen module.

Impact:
- Feature-driven changes to a shared audit utility
- Policy requires revert or a dedicated scoped ticket

Recommendation:
Revert `audit-log.utils.ts`. Build the crew audit label map in the consumer / feature constants from `FORM_SECTIONS` (flatten `fields` or `leftFields`+`rightFields` locally).
---

---
SkillShow Users path migration may 404 without permission route update

Risk Level: HIGH
File Path: src/pages/management/skillshow-users/breadcrumb.ts
Lines: 5

Description:
**Contract / Global consistency** — `TEAM_USERS_LIST_PATH` moves from `/user-management/skillshow-users` to `/settings/skillshow-users`, with legacy redirects. Routes are permission-driven; neither #348 nor backend #243 updates permission seed/DB routes. If permissions still register the old path, menu → redirect → new URL can 404.

Impact:
- Broken menu / deep links after deploy if permission tree is not migrated in the same release

Recommendation:
Confirm (or ship) permission catalogue route `/settings/skillshow-users` before merge. Smoke-test menu, legacy URL, and `/settings/skillshow-users/add`.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed: `audit-log.utils.ts` | CRITICAL | Open | `src/utils/audit-log.utils.ts` | 5-21 |
| 2 | Path migration may 404 without permission route update | HIGH | Open | `src/pages/management/skillshow-users/breadcrumb.ts` | 5 |

**Merge readiness:** Not ready — 1 Critical (protected module) + 1 High (permission path).
