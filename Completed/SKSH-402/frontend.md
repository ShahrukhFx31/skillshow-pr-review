# PR review (SKSH-402) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#348](https://github.com/SkillshowFx/skillshow-admin-ui/pull/348) |
| Branch | `SKSH-402` → `main` |
| Head | `272865a20085922a18403729055cfb0541ff0d7e` |
| Scope | Crew list columns/sort; SkillShow Users path → `/settings/...`; crew onboarding skip/branding; audit labels |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Verified | 2026-07-17 vs prior head `3b5ed29` |

### Protected modules

| Module | Status |
|--------|--------|
| `src/utils/audit-log.utils.ts` | **Not modified** vs `main` ✅ (prior edit reverted) |
| Pagination / destructive-confirm | **Not modified** ✅ |

### Positive notes

- Crew list columns/sort keys align with backend #243.
- Legacy redirect + runtime `permission-route-remap.ts` keep menu/routes on `/settings/skillshow-users`.
- Crew audit labels built in feature constants (`CREW_USER_AUDIT_FIELD_LABELS` from `FORM_SECTIONS`).

## GitHub comments

No open findings to post (prior REQUEST_CHANGES resolved on head).

## Findings

---
Protected module changed: `audit-log.utils.ts`

Risk Level: CRITICAL
File Path: src/utils/audit-log.utils.ts
Lines: 5-21

Description:
**Protected module** — PR had extended `buildAuditFieldLabelMap` for two-column form sections.

Impact:
- Feature-driven changes to a frozen shared utility

Recommendation:
Revert; build the crew audit label map in feature constants.

Status: ✅ Fixed — `audit-log.utils.ts` matches `main` (0-byte diff); crew uses `CREW_USER_AUDIT_FIELD_LABELS` in onboarding constants.
---

---
SkillShow Users path migration may 404 without permission route update

Risk Level: HIGH
File Path: src/pages/management/skillshow-users/breadcrumb.ts
Lines: 5

Description:
**Contract / Global consistency** — Path moved to `/settings/skillshow-users` without permission catalogue update.

Impact:
- Menu / deep links could 404 if DB still had `user-management/...`

Recommendation:
Ship permission route update or equivalent remap before merge.

Status: ✅ Fixed — `permission-route-remap.ts` + wire-up in `use-permission-routes.tsx`; `PERMISSIONS.md` documents DB migration preference.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed: `audit-log.utils.ts` | CRITICAL | ✅ Fixed | `src/utils/audit-log.utils.ts` | 5-21 |
| 2 | Path migration may 404 without permission route update | HIGH | ✅ Fixed | `skillshow-users/breadcrumb.ts` | 5 |

**Merge readiness:** Ready — prior Critical/High findings fixed on `272865a2`; no new Critical/High/Medium open.
