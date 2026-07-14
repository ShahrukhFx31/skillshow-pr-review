# PR review (SKSH-399) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#339](https://github.com/SkillshowFx/skillshow-admin-ui/pull/339) |
| Branch | `SKSH-399` → `main` |
| Head | `ac02f87403a5fa7e3fe1bed647ecbe202ab47f37` |
| Scope | Split App Users into Athletes / Parents / Coaches with role-scoped routes |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

### Protected modules

| Module | Status |
|--------|--------|
| pagination-bar, use-pagination, use-server-table-controls, table-sort, audit-log, DestructiveActionConfirmModal | **Not modified** ✅ |

### Positive notes

- `AppUserRolePage` + thin Vite entries; permission paths documented.
- Server list keeps `useServerTableControls`; role in `extraFilterState` resets page.
- Delete still uses `DestructiveActionConfirmModal`.
- Legacy redirects (app-users → athletes, teams → coaches) are thoughtful.

## GitHub comments

### `src/pages/management/app-users/onboarding/AppUserOnboardingPage.tsx`

- **L53** — Role-scoped routes do not verify detail `role` matches page role (HIGH)

### `src/pages/management/app-users/onboarding/components/app-user-form.tsx`

- **L231** — Hidden role field uses ternary-null (MEDIUM)

### `src/pages/management/app-users/dashboard/components/app-users-columns.tsx`

- **L163** — CSV export still bound to athlete-shaped `APP_USER_COLUMNS` (MEDIUM)

### `src/pages/management/app-users/breadcrumb.ts`

- **L75** — Legacy `/add` path matching is overly broad (MEDIUM)

## Findings

---
Role-scoped routes do not verify detail `role` matches page role

Risk Level: HIGH
File Path: src/pages/management/app-users/onboarding/AppUserOnboardingPage.tsx
Lines: 53

Description:
**Contract** / routing — Loads by `userId` only (`appUserDetailMatchesRoute` checks `seq`). No guard that `activeDetail.role ===` scoped `role`. Manual URL (`/user-management/coaches?mode=view&userId=<athleteSeq>`) can open another role under the wrong permission entry. Teams already guards `detail.role === "coach"`.

Impact:
- Wrong role menus show/edit the wrong user type.
- Breadcrumbs/nav stay on the wrong list after save.

Recommendation:
After detail loads, if `activeDetail.role !== role`, redirect to `appUserActionPath(activeDetail.role, mode, userId)` (or back to scoped list).
---

---
Hidden role field uses ternary-null

Risk Level: MEDIUM
File Path: src/pages/management/app-users/onboarding/components/app-user-form.tsx
Lines: 231

Description:
**JSX conditional rendering** — `{isAddMode ? <Form.Item hidden name="role" /> : null}`.

Impact:
- Style inconsistency with project JSX rule.

Recommendation:
Prefer `{isAddMode && <Form.Item hidden name="role" />}`.
---

---
CSV export still bound to athlete-shaped `APP_USER_COLUMNS`

Risk Level: MEDIUM
File Path: src/pages/management/app-users/dashboard/components/app-users-columns.tsx
Lines: 163

Description:
**DRY** / **Global consistency** — Deprecated `APP_USER_COLUMNS = getAppUserColumns(APP_USER_ROLE.athlete)` remains the export source. Role-scoped dashboards still export via that static shape.

Impact:
- Export naming/shape not role-aware; fragile if columns diverge by role later.

Recommendation:
Pass role into export (`getAppUserColumns(role)`, role-based filename), or drop deprecated `APP_USER_COLUMNS`.
---

---
Legacy `/add` path matching is overly broad

Risk Level: MEDIUM
File Path: src/pages/management/app-users/breadcrumb.ts
Lines: 75

Description:
`pathname.endsWith("/add") || pathname.includes("/add")` — `includes` makes `endsWith` redundant and can false-match segments containing `"add"`.

Impact:
- Unexpected redirects on oddly named legacy paths.

Recommendation:
Match segment boundaries only, e.g. `pathname === \`${list}/add\`` or `/\/add\/?$/`.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Role-scoped routes do not verify detail `role` matches page role | HIGH | Open | `src/pages/management/app-users/onboarding/AppUserOnboardingPage.tsx` | 53 |
| 2 | Hidden role field uses ternary-null | MEDIUM | Open | `src/pages/management/app-users/onboarding/components/app-user-form.tsx` | 231 |
| 3 | CSV export still bound to athlete-shaped `APP_USER_COLUMNS` | MEDIUM | Open | `src/pages/management/app-users/dashboard/components/app-users-columns.tsx` | 163 |
| 4 | Legacy `/add` path matching is overly broad | MEDIUM | Open | `src/pages/management/app-users/breadcrumb.ts` | 75 |

**Merge readiness:** Not ready — 1 open High finding.
