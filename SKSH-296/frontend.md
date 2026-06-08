# Frontend PR Review — skillshow-admin-ui (`SKSH-296`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSK-296` (remote/local; ticket id `SKSH-296`)  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-08 @ `802fba45`  
**Scope:** SkillShow user audit log UI — shared `AuditLog`, team-user view integration, removal of legacy modification fields (Critical, High, Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract enforced)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 3 (0 Critical, 0 High open, 1 Medium) — **2 Fixed**, **1 Accepted**

> **Branch note:** Local/remote branch is named `SKSK-296`; Jira ticket is `SKSH-296`. Audit-log core: `9492bfec`; bug-fix pass: `802fba45` (typed detail row, cache invalidation, `AuditLog` component).

### Protected modules

No changes to `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `antd.adapter.tsx`, or `destructive-action-confirm-modal.tsx`.

---

## GitHub comments (Open findings)

None.

---

---
Detail API response type omits `history`

Risk Level: HIGH  
File Path: src/api/services/skillshowUserService.ts  
Lines: 23-26

Description:
**Contract / DRY.** Backend detail returns `history[]`. Initial PR cast around `TeamUserRow`; fix commit adds `TeamUserDetailRow` and types `getSkillshowUser` accordingly. `TeamUserAuditLog` reads `data?.history` without a cast.

Impact:
- TypeScript now enforces `history` on detail responses

Recommendation:
N/A — implemented.

**Re-review (`802fba45`):** ✅ **Fixed** — `TeamUserDetailRow` in `dashboard/types.ts`; `getSkillshowUser` returns `Promise<TeamUserDetailRow>`; `team-user-audit-log.tsx` line 19 uses `data?.history ?? []`.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Detail query cache not invalidated after team-user mutations

Risk Level: HIGH  
File Path: src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx  
Lines: 148-155

Description:
**Contract.** `patchTeamUserMutation` now invalidates `teamUserDetailQueryKey(userId)` immediately and again after `AUDIT_LOG_REFETCH_DELAY_MS` (1500 ms) to account for backend CDC debounce. Shared `teamUserDetailQueryKey` is used in onboarding page and audit log.

Impact:
- Audit log refreshes after save instead of serving stale React Query cache
- Delayed second invalidation improves odds CDC history is persisted before refetch

Recommendation:
N/A — implemented.

**Re-review (`802fba45`):** ✅ **Fixed** — lines 150-154 invalidate detail key + delayed refetch; `teamUserDetailQueryKey` centralized in `constants.ts`.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Shared audit component path vs frozen module name

Risk Level: MEDIUM  
File Path: src/components/AuditLog.tsx  
Lines: 1

Description:
**Global consistency / Protected module.** Review prompt documents `src/components/AuditLogTable.tsx`. Fix commit moved shared UI to `src/components/AuditLog.tsx` (renamed from `audit-log/AuditLogTable.tsx`).

Impact:
- Shared component is at the correct top-level `src/components/` placement
- Export name `AuditLog` vs prompt alias `AuditLogTable` is documentation-only drift

Recommendation:
Optional: add `AuditLogTable.tsx` re-export alias for prompt parity. Not required for merge.

**Accepted (2026-06-08):** Team standardized on `src/components/AuditLog.tsx`; placement matches shared-component convention. Prompt frozen-path name can catch up separately.

**Status:** Accepted

---

## Additional notes (not blockers)

- **`TeamUserAuditLog` shares `teamUserDetailQueryKey` with parent** — React Query dedupes network calls; acceptable.
- **`AuditLog` is client-side only** (embedded `history[]` on detail) — no `useServerTableControls` required.
- **Description rendering** aligns with backend `buildDescription` clause shapes.
- **Legacy `modificationOn` / `modificationBy` removed** from types, constants, and form props.

---

## Positive notes

- Replaces static created/modified `Descriptions` with a proper event timeline.
- `AuditLog` provides responsive mobile cards + desktop `Table` with shared formatting.
- `hideWhenEmpty` avoids empty audit cards on new users.
- Typed detail contract + CDC-aware cache invalidation close the main frontend gaps.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Detail API response type omits `history` | HIGH | ✅ Fixed | src/api/services/skillshowUserService.ts | 23-26 |
| 2 | Detail query cache not invalidated after team-user mutations | HIGH | ✅ Fixed | src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx | 148-155 |
| 3 | Shared audit component path vs frozen module name | MEDIUM | Accepted | src/components/AuditLog.tsx | 1 |

**Merge readiness:** **Merge-ready (frontend)** — no open High/Medium blockers. Backend also clear — all findings Fixed or Accepted (see [backend.md](./backend.md)).
