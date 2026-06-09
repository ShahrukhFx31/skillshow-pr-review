# Frontend PR Review — skillshow-admin-ui (`SKSH-296`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-296-1`  
**Base:** `main...HEAD` @ `1110cd39`  
**Initial review:** 2026-06-09  
**Scope:** Shared audit-log UI (`UserAuditLogPanel`, `AuditLogDescription`), per-entity audit panels on view forms, API hooks, query invalidation on patch/bulk/logo upload; removal of legacy `modificationOn` / `modificationBy` from crew/skillshow types (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 2 (0 Critical, 1 High, 1 Medium) — **2 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `AuditLogTable.tsx`, `pagination-bar.tsx`, `use-server-table-controls.ts`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx` | **Not modified** (and `AuditLogTable` does not exist on branch — new `UserAuditLogPanel` is appropriate for embedded entity history) |

Entity audit panels load a fixed API slice (no client pagination) — **Accepted**; not a server-driven admin list page.

---

## GitHub comments (Open findings)

| # | File (inline comment anchor) | PR comment line | Gap file (not in PR diff) |
|---|------------------------------|-----------------|---------------------------|
| 1 | `src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx` | 150 | `team-users-bulk-update.tsx` |
| 2 | `src/pages/management/crew-users/onboarding/components/crew-user-audit-log.tsx` | 11 | — |

---

## Findings

---
Team users bulk update does not invalidate audit-log queries

Risk Level: HIGH
File Path: src/pages/management/skillshow-users/dashboard/components/team-users-bulk-update.tsx
Lines: 20-25

**Not in PR diff.** `team-users-bulk-update.tsx` is unchanged on `SKSH-296-1` (only `app-users-bulk-update.tsx` and `partners-bulk-update.tsx` gained audit invalidation). The gap is real but GitHub cannot attach an inline comment on that file — use the anchor below on a changed sibling file, or add a general PR comment.

Description:
**Contract / stale data / Global consistency.** `app-users-bulk-update.tsx` and `partners-bulk-update.tsx` (both **changed in this PR**) invalidate per-entity audit query keys after bulk patch. `team-users-bulk-update.tsx` (**unchanged**) only calls `invalidateQueries({ queryKey: TEAM_USERS_LIST_QUERY_KEY })`, so `teamUserAuditLogQueryKey(userId)` caches remain stale after bulk status/department changes. `team-user-form.tsx` **does** invalidate audit on patch (line 150) — bulk path was missed.

Impact:
- Admin viewing a team user's audit panel after a bulk update from the dashboard sees outdated entries until manual navigation refresh.
- Inconsistent behavior vs app-user and partner bulk flows in the same PR.

Recommendation:
Match the app-user pattern:

```typescript
onSuccess: (_rows, variables) => {
  queryClient.invalidateQueries({ queryKey: TEAM_USERS_LIST_QUERY_KEY });
  for (const userId of variables.userIds) {
    void queryClient.invalidateQueries({ queryKey: teamUserAuditLogQueryKey(userId) });
  }
  onBulkComplete();
},
```

Note: `crew-users-bulk-update.tsx` uses a broad predicate (`management` + `crew-users`) that already covers `crewUserAuditLogQueryKey` — no change needed there.

**PR comment (line 150, `team-user-form.tsx`):** **High:** Patch invalidates `teamUserAuditLogQueryKey` here, but `team-users-bulk-update.tsx` (not in this PR diff) does not — bulk status/department updates leave audit caches stale. Add the same per-`userId` invalidation loop as `app-users-bulk-update.tsx` line 24–25.

**Alternative anchor (`app-users-bulk-update.tsx` line 25, in diff):** **High:** Team users bulk update should mirror this audit invalidation; `team-users-bulk-update.tsx` was not updated in this PR.
---

---
Duplicate per-entity audit-log wrapper components

Risk Level: MEDIUM
File Path: src/pages/management/crew-users/onboarding/components/crew-user-audit-log.tsx
Lines: 1-25

Description:
**DRY.** `crew-user-audit-log.tsx`, `team-user-audit-log.tsx`, `app-user-audit-log.tsx`, and `partner-audit-log.tsx` are thin duplicates: `useQuery` + `UserAuditLogPanel` with only API function, query key, and `fieldLabels` varying.

Impact:
- Future audit-panel behavior changes (loading skeleton, empty state, error handling) require four edits.
- Higher chance of inconsistent invalidation or `enabled` guards across entities.

Recommendation:
Extract a shared hook or component:

```typescript
export function useEntityAuditLogs<T extends AuditLogEntry>(
  queryKey: readonly unknown[],
  queryFn: () => Promise<T[]>,
  enabled: boolean,
) { /* useQuery wrapper */ }

export function EntityAuditLogCard(props: {
  userId: string;
  queryKey: (id: string) => readonly unknown[];
  listFn: (id: string) => Promise<AuditLogEntry[]>;
  fieldLabels: Record<string, string>;
  cardClassName: string;
}) { /* ... */ }
```

Keep entity files as one-liner re-exports with constants only.

**PR comment (line 11):** **Medium (DRY):** Four near-identical audit wrappers differ only in API fn, query key, and field labels. Consider a shared `useEntityAuditLogs` hook or `EntityAuditLogPanel` factory so future panel behavior changes land in one place.
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Team users bulk update does not invalidate audit-log queries | HIGH | Open | src/pages/management/skillshow-users/dashboard/components/team-users-bulk-update.tsx | 20-25 | 150 (`team-user-form.tsx`; gap file not in diff) |
| 2 | Duplicate per-entity audit-log wrapper components | MEDIUM | Open | src/pages/management/crew-users/onboarding/components/crew-user-audit-log.tsx | 1-25 | 11 |

### Positive notes

- Shared presentation: `UserAuditLogPanel` + `AuditLogDescription` with responsive desktop table / mobile `Descriptions`.
- `buildAuditFieldLabelMap` reuses form section labels — single source for field display names.
- Patch mutations invalidate audit queries for app-user, crew-user, team-user (form), partner form, and partner logo upload.
- Legacy `modificationOn` / `modificationBy` removed from crew/skillshow types and forms; audit panel replaces embedded modification display.
- `formatAuditDisplayValue` normalizes status labels for readable audit text.

**Merge readiness:** **1 open High blocker** (#1 stale audit after team bulk update). Medium #2 is recommended cleanup, not a merge blocker per team convention.
