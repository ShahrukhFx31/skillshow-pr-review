# Frontend PR Review — skillshow-admin-ui (`SKSH-348`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-348`  
**Base:** `main...HEAD` @ `60495c6e`  
**Initial review:** 2026-06-16  
**Scope:** Parent My Athlete view-profile Teams tab (coach teams API), API client proactive token refresh + hydration gate (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 1 (0 Critical, 0 High, 1 Medium) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/myAthlete/viewProfile/index.tsx` | Parent-only Teams tab; URL guard when tab hidden |
| `src/pages/myAthlete/viewProfile/components/teams-tab/AthleteTeamsTab.tsx` | `useQuery` + teams grid / empty / error states |
| `src/pages/myAthlete/viewProfile/components/teams-tab/LinkedAthleteTeamCard.tsx` | Team card + skeleton |
| `src/api/services/athleteService.ts` | `getLinkedAthleteTeams` |
| `src/api/types/athlete.types.ts` | `LinkedAthleteTeamItemDto` / `LinkedAthleteTeamsRes` |
| `src/types/athlete-view-profile.types.ts` | `AthleteTeamsTabProps` → `relationId` |
| `src/api/apiClient.ts` | Hydration gate, proactive refresh, shared `attachAuthorizationHeader` |
| `src/api/utils/auth-token.utils.ts` | **New** — `isAccessTokenExpiredOrNearExpiry` |
| `src/api/utils/user-store-hydration.ts` | **New** — `waitForUserStoreHydration` |
| `src/api/constants/client.ts` | `ACCESS_TOKEN_REFRESH_BUFFER_SECONDS` |
| `.husky/pre-commit`, `package.json` | lint-staged tightened; full `pnpm lint` removed from hook |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Teams tab is a read-only card grid (not a server list) — no `useServerTableControls` required | ✅ Contract |
| Reuses `VIEW_PROFILE_QUERY_OPTIONS` (same stale/refetch policy as profile hooks) | ✅ |
| `inferAddConnectionMode` + `AddConnectionMode.Parent` for parent-only tab | ✅ |
| Redirects `?tab=teams` when tab hidden (coach viewer) | ✅ |
| API client: deduped `refreshPromise`, skips refresh on public/refresh URLs | ✅ |
| Query key not added to `ATHLETE_QUERY_KEYS` | ⚠️ See finding #1 |
| Husky: lint via lint-staged only | ✅ Accepted (tooling) |

### Positive notes

- **API client:** `waitForUserStoreHydration` + proactive refresh addresses race where the Teams tab fired before persisted tokens were restored (likely motivator for the second commit).
- **UI:** `LinkedAthleteTeamCard` handles logo fallback, skeleton, and empty/error states cleanly.
- **Types:** Frontend DTO mirrors backend `LinkedAthleteTeamItem` field-for-field.

---

## GitHub comments

**`src/pages/myAthlete/viewProfile/components/teams-tab/AthleteTeamsTab.tsx` (line 10)**  
Teams uses a local `LINKED_ATHLETE_TEAMS_QUERY_KEY` string while sibling view-profile queries use `ATHLETE_QUERY_KEYS` in `athlete.constants.ts`. Please add `linkedTeams(relationId)` there and use it for `queryKey` so future invalidation (e.g. after roster changes) stays consistent with `completeProfile` / `linkedAccountGeneral`.

---

## Findings

---
Teams query key bypasses ATHLETE_QUERY_KEYS

Risk Level: MEDIUM  
File Path: skillshow-admin-ui/src/pages/myAthlete/viewProfile/components/teams-tab/AthleteTeamsTab.tsx  
Lines: 10-16

Description:
**DRY / Global consistency.** `AthleteTeamsTab` defines `LINKED_ATHLETE_TEAMS_QUERY_KEY = "linked-athlete-teams"` inline, while `useAthleteCompleteProfile` and other athlete view-profile hooks use centralized keys from `ATHLETE_QUERY_KEYS` (`completeProfile`, `linkedAccountGeneral`, etc.) documented as the single source for invalidation.

Impact:
- Future cache invalidation after team membership changes will be easy to miss or implement with a mismatched key string.
- Inconsistent pattern across the same feature folder (`viewProfile/hooks` vs teams tab).

Recommendation:
Add to `src/constants/athlete.constants.ts`:

```ts
linkedTeams: (relationId: string) => ["athletes", "linked-teams", relationId] as const,
```

Use `ATHLETE_QUERY_KEYS.linkedTeams(relationId)` in `AthleteTeamsTab` (and any future invalidation call sites).

**GitHub comment:** Teams tab uses a one-off query key string; please add `linkedTeams(relationId)` to `ATHLETE_QUERY_KEYS` and use it here so invalidation matches the rest of the view-profile queries.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Teams query key bypasses ATHLETE_QUERY_KEYS | MEDIUM | Open | skillshow-admin-ui/src/pages/myAthlete/viewProfile/components/teams-tab/AthleteTeamsTab.tsx | 10-16 |

**Merge readiness:** No Critical or High blockers. One Medium DRY follow-up on query-key centralization — acceptable to merge if deferred, but mark **Accepted** or fix before archiving to `Completed/`.
