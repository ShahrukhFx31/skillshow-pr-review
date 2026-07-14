# PR review (SKSH-399) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#339](https://github.com/SkillshowFx/skillshow-admin-ui/pull/339) |
| Branch | `SKSH-399` → `main` |
| Head | `97903355d3b47f9c345cdbd88db3d79caf52170f` |
| Scope | Split App Users into Athletes / Parents / Coaches with role-scoped routes |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-14 — head `97903355` (role guard, export by role, `/add` match, JSX `&&`) |

### Protected modules

| Module | Status |
|--------|--------|
| pagination-bar, use-pagination, use-server-table-controls, table-sort, audit-log, DestructiveActionConfirmModal | **Not modified** ✅ |

### Positive notes

- `AppUserRolePage` + thin Vite entries; permission paths documented.
- Server list keeps `useServerTableControls`; role in `extraFilterState` resets page.
- Delete still uses `DestructiveActionConfirmModal`.
- Role mismatch redirects to scoped list; export uses `getAppUserColumns(role)`.

## GitHub comments

_No open findings._

## Findings

---
Role-scoped routes do not verify detail `role` matches page role

Risk Level: HIGH
File Path: src/pages/management/app-users/onboarding/AppUserOnboardingPage.tsx
Lines: 53

Description:
Load matched `seq` only; wrong role URL could open another user type.

**Re-verify (97903355):** ✅ Fixed — `routeMatchedDetail.role === role` gate; `useEffect` redirects to scoped list on mismatch; `activeDetail` only set when roles match.
---

---
Hidden role field uses ternary-null

Risk Level: MEDIUM
File Path: src/pages/management/app-users/onboarding/components/app-user-form.tsx
Lines: 231

Description:
`isAddMode ? … : null`.

**Re-verify (97903355):** ✅ Fixed — `isAddMode && <Form.Item hidden name="role" />`.
---

---
CSV export still bound to athlete-shaped `APP_USER_COLUMNS`

Risk Level: MEDIUM
File Path: src/pages/management/app-users/dashboard/components/app-users-columns.tsx
Lines: 163

Description:
Static athlete columns for CSV.

**Re-verify (97903355):** ✅ Fixed — `mapAppUsersToCsvRows` / `exportAppUsersRowsAsCsv` take `role`; dashboard passes scoped role; deprecated `APP_USER_COLUMNS` removed.
---

---
Legacy `/add` path matching is overly broad

Risk Level: MEDIUM
File Path: src/pages/management/app-users/breadcrumb.ts
Lines: 75

Description:
`pathname.includes("/add")` could false-match.

**Re-verify (97903355):** ✅ Fixed — `pathname === \`${list}/add\` || pathname.endsWith("/add")` (removed `includes`).
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Role-scoped routes do not verify detail `role` matches page role | HIGH | ✅ Fixed | `src/pages/management/app-users/onboarding/AppUserOnboardingPage.tsx` | 53 |
| 2 | Hidden role field uses ternary-null | MEDIUM | ✅ Fixed | `src/pages/management/app-users/onboarding/components/app-user-form.tsx` | 231 |
| 3 | CSV export still bound to athlete-shaped `APP_USER_COLUMNS` | MEDIUM | ✅ Fixed | `src/pages/management/app-users/dashboard/components/app-users-columns.tsx` | 163 |
| 4 | Legacy `/add` path matching is overly broad | MEDIUM | ✅ Fixed | `src/pages/management/app-users/breadcrumb.ts` | 75 |

**Merge readiness:** No open Critical/High/Medium blockers.
