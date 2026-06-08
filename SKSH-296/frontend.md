# Frontend PR Review — skillshow-admin-ui (`SKSH-296`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSK-296` (remote/local; ticket id `SKSH-296`)  
**Base:** `main...HEAD` (feature commit `9492bfec`)  
**Scope:** SkillShow user audit log UI — shared `AuditLogTable`, team-user view integration, removal of legacy modification fields (Critical, High, Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract enforced)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 3 (0 Critical, 2 High, 1 Medium) — **3 Open**

> **Branch note:** Local/remote branch is named `SKSK-296`; Jira ticket is `SKSH-296`. Branch tip includes merged `SKSH-311` commits; audit-log scope is isolated to `9492bfec` (7 files).

---

## GitHub comments (Open findings)

### 1. `src/api/services/skillshowUserService.ts` line 22

**High:** `getSkillshowUser` is still typed as `Promise<TeamUserRow>` but the API now returns `history[]` on detail — please add `TeamUserDetailRow` (or extend the row type) so consumers don't need unsafe casts.

### 2. `src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx` lines 147-149

**High:** `patchTeamUserMutation` only invalidates `TEAM_USERS_LIST_QUERY_KEY` after save — please also invalidate `["management", "team-users", "detail", userId]` (and consider a short delayed refetch) so the audit log picks up CDC-written history after edit.

### 3. `src/components/audit-log/AuditLogTable.tsx` line 1

**Medium:** New shared audit table lives at `audit-log/AuditLogTable.tsx` while the frozen contract documents `src/components/AuditLogTable.tsx` — align path before crew/app users migrate to the same component.

---

---
Detail API response type omits `history`

Risk Level: HIGH  
File Path: src/api/services/skillshowUserService.ts  
Lines: 22-25

Description:
**Contract / DRY.** Backend `getSkillshowUser` now returns `SkillshowUserDetailRow` with `history: SkillshowUserHistoryEntryDto[]`. Frontend `getSkillshowUser` still returns `Promise<TeamUserRow>`, and `TeamUserAuditLog` casts with a local `TeamUserDetailWithHistory` type instead of a shared API type.

Impact:
- TypeScript will not catch missing or mis-shaped `history` fields at compile time
- Other consumers of `getSkillshowUser` won't know `history` exists without reading backend docs

Recommendation:
Add a typed detail row and use it in the service + feature types:

```typescript
export type TeamUserHistoryEntry = {
  id: string;
  createdAt: string;
  actor: TeamUserActorRef;
  description: string;
};

export type TeamUserDetailRow = TeamUserRow & { history: TeamUserHistoryEntry[] };

export function getSkillshowUser(userId: string): Promise<TeamUserDetailRow> {
  return apiClient.get<TeamUserDetailRow>({ ... });
}
```

Remove the cast in `team-user-audit-log.tsx`.

**PR comment (line 22):**  
**High:** `getSkillshowUser` is still typed as `Promise<TeamUserRow>` but the API now returns `history[]` on detail — please add `TeamUserDetailRow` (or extend the row type) so consumers don't need unsafe casts.

**Status:** Open

---

---
Detail query cache not invalidated after team-user mutations

Risk Level: HIGH  
File Path: src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx  
Lines: 143-150

Description:
**Contract.** After a successful patch, `patchTeamUserMutation` invalidates only `TEAM_USERS_LIST_QUERY_KEY` and navigates to view. The detail query `["management", "team-users", "detail", userId]` used by `TeamUserOnboardingPage` and `TeamUserAuditLog` is left cached. Combined with backend async CDC (1s debounce), the audit table can show stale or missing entries immediately after save.

Impact:
- User saves edits → view page may show old audit log from React Query cache
- Even on refetch, history may not include the just-saved change until CDC flushes (~1s later) with no follow-up refetch

Recommendation:
Invalidate detail on mutation success (and after resend-welcome / bulk flows if applicable):

```typescript
onSuccess: (_row, { userId }) => {
  queryClient.invalidateQueries({ queryKey: TEAM_USERS_LIST_QUERY_KEY });
  queryClient.invalidateQueries({ queryKey: ["management", "team-users", "detail", userId] });
  navigateTeamUser(navigate, teamUserRoutes.view(userId));
},
```

Optionally `refetch` detail after a short delay or return appended history from PATCH for synchronous UI.

**PR comment (line 148):**  
**High:** `patchTeamUserMutation` only invalidates `TEAM_USERS_LIST_QUERY_KEY` after save — please also invalidate `["management", "team-users", "detail", userId]` (and consider a short delayed refetch) so the audit log picks up CDC-written history after edit.

**Status:** Open

---

---
Shared AuditLogTable path does not match frozen module location

Risk Level: MEDIUM  
File Path: src/components/audit-log/AuditLogTable.tsx  
Lines: 1

Description:
**Global consistency / Protected module.** Review contract lists `src/components/AuditLogTable.tsx` as the frozen shared audit table. This PR introduces `src/components/audit-log/AuditLogTable.tsx` instead. `crew-user-audit-log.tsx` still uses legacy `Descriptions` markup and will need migration later.

Impact:
- Future crew/app-user audit migrations may import the wrong path or duplicate table markup
- Drift from documented protected-module location

Recommendation:
Move/re-export from `src/components/AuditLogTable.tsx` (re-export from `audit-log/` if subfolder is preferred), or update the team standard before additional features adopt the component.

**PR comment (line 1):**  
**Medium:** New shared audit table lives at `audit-log/AuditLogTable.tsx` while the frozen contract documents `src/components/AuditLogTable.tsx` — align path before crew/app users migrate to the same component.

**Status:** Open

---

## Additional notes (not blockers)

- **`TeamUserAuditLog` shares queryKey with parent** (`["management", "team-users", "detail", userId]`) — React Query dedupes network calls; acceptable, though passing `history` from the parent would simplify the child.
- **`AuditLogTable` is client-side only** (embedded `history[]` on detail) — no `useServerTableControls` required for this slice.
- **Description rendering** aligns with backend `buildDescription` clause shapes (`set to`, `changed from`, `was cleared`).
- **Legacy `modificationOn` / `modificationBy` removed** from types, constants, and form props — consistent with backend schema change.

---

## Positive notes

- Replaces static created/modified `Descriptions` with a proper event timeline.
- `AuditLogTable` provides responsive mobile cards + desktop `Table` with shared formatting.
- `hideWhenEmpty` avoids empty audit cards on new users.
- Form simplification — audit section self-loads in view mode via `routeUserId`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Detail API response type omits `history` | HIGH | Open | src/api/services/skillshowUserService.ts | 22-25 |
| 2 | Detail query cache not invalidated after team-user mutations | HIGH | Open | src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx | 143-150 |
| 3 | Shared AuditLogTable path does not match frozen module location | MEDIUM | Open | src/components/audit-log/AuditLogTable.tsx | 1 |

**Merge readiness:** **Not merge-ready** — 2 High Open findings (type contract + stale audit cache after save); address alongside backend CDC timing. Medium path alignment recommended before wider audit-log rollout.
