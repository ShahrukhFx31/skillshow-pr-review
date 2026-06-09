# Frontend PR Review — skillshow-admin-ui (`SKSH-296`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-296-1`  
**Base:** `main...HEAD` @ `b0f9c9a9`  
**Initial review:** 2026-06-09  
**Re-reviewed:** 2026-06-09 — fixes verified on branch (`c6142097` `fix: improvement and fixes`)  
**Scope:** Shared audit-log UI (`EntityAuditLogCard`, `UserAuditLogPanel`, `AuditLogDescription`), per-entity audit panels, API hooks, query invalidation on patch/bulk/logo upload; removal of legacy `modificationOn` / `modificationBy` from crew/skillshow types (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 2 (0 Critical, 1 High, 1 Medium) — **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `AuditLogTable.tsx`, `pagination-bar.tsx`, `use-server-table-controls.ts`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx` | **Not modified** (`AuditLogTable` does not exist — `EntityAuditLogCard` + `UserAuditLogPanel` used for embedded entity history) |

Entity audit panels load a fixed API slice (no client pagination) — **Accepted**.

---

## GitHub comments (Open findings)

*None — all findings Fixed.*

---

## Findings

---
Team users bulk update does not invalidate audit-log queries

Risk Level: HIGH
File Path: src/pages/management/skillshow-users/dashboard/components/team-users-bulk-update.tsx
Lines: 23-28

Description:
**Contract / stale data / Global consistency.** Initial review: `team-users-bulk-update.tsx` did not invalidate `teamUserAuditLogQueryKey` after bulk patch while app-user and partner bulk flows did.

Impact:
- Stale audit panel after bulk status/department updates until manual refresh.

Recommendation:
Invalidate per-`userId` audit query keys in bulk `onSuccess`, matching app-user pattern.

**PR comment (line 26):** **High:** Bulk updates must invalidate `teamUserAuditLogQueryKey(userId)` for each selected user.

**Re-review (`c6142097`):** **Fixed** — `team-users-bulk-update.tsx` imports `teamUserAuditLogQueryKey` and invalidates per `variables.userIds` in `onSuccess` (lines 25–27).
---

---
Duplicate per-entity audit-log wrapper components

Risk Level: MEDIUM
File Path: src/components/audit-log/entity-audit-log-card.tsx
Lines: 1-34

Description:
**DRY.** Four near-identical entity audit wrappers duplicated `useQuery` + `UserAuditLogPanel` wiring.

Impact:
- Four edit sites for panel behavior changes.

Recommendation:
Shared `EntityAuditLogCard` with `listAuditLogs`, `queryKey`, and `fieldLabels` props.

**PR comment (line 20):** **Medium (DRY):** Extract shared entity audit card — four wrappers differ only in API fn, query key, and labels.

**Re-review (`c6142097`):** **Fixed** — `EntityAuditLogCard` centralizes query + panel; `app-user-audit-log.tsx`, `crew-user-audit-log.tsx`, `team-user-audit-log.tsx`, and `partner-audit-log.tsx` are thin composition layers.
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Team users bulk update does not invalidate audit-log queries | HIGH | ✅ Fixed | src/pages/management/skillshow-users/dashboard/components/team-users-bulk-update.tsx | 23-28 | 26 |
| 2 | Duplicate per-entity audit-log wrapper components | MEDIUM | ✅ Fixed | src/components/audit-log/entity-audit-log-card.tsx | 1-34 | 20 |

### Re-review notes (2026-06-09)

| Change | Verdict |
|--------|---------|
| `team-users-bulk-update.tsx` per-user audit invalidation | Fixes #1 |
| `EntityAuditLogCard` + thin entity wrappers | Fixes #2 |
| `crew-users-bulk-update.tsx` broad predicate still covers `crewUserAuditLogQueryKey` | OK |

### Positive notes

- Shared `EntityAuditLogCard` → `UserAuditLogPanel` → `AuditLogDescription` with responsive desktop/mobile layouts.
- `buildAuditFieldLabelMap` reuses form section labels.
- Audit query invalidation on patch (all entities), bulk (app-user, partner, team-user, crew via predicate), and partner logo upload.
- Legacy `modificationOn` / `modificationBy` removed from crew/skillshow types and forms.

**Merge readiness:** **No open Critical/High/Medium blockers on the frontend diff.**
