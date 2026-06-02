# Frontend PR Review — skillshow-admin-ui (`SKSH-137`)

**Base:** `main...HEAD`  
**Branch:** `SKSH-137`  
**Reviewed:** 2026-05-26  
**Scope:** SkillShow team user management UI, partner dashboard/delete/connect message, legacy `/management/user` removal  
**Findings:** 1 open (**0 Critical**, **1 High**)

---

## Mutations lack user-visible error handling

---
Mutations lack user-visible error handling

Risk Level: HIGH
File Path: src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx
Lines: 133-174

Description:
`createTeamUserMutation` and `patchTeamUserMutation` only define `onSuccess` handlers. There is no `onError` (or global mutation cache handler) to show toast/field errors when the API returns 400/500.

Impact:
- Failed create/update appears to do nothing after clicking Save
- Admins cannot distinguish validation errors from network failures without opening devtools

Recommendation:
Add `onError` with `toast.error` (or map API error body to Ant `Form` fields), consistent with `useTeamUserActions` resend (`toast.promise`) and other management pages.

**PR comment (lines 133-174):** **High:** Create/patch mutations have no `onError`—failed saves fail silently. Add toast or form-level error mapping so admins see API validation failures.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Silent mutation failures | HIGH | Open | `team-user-form.tsx` | 133-174 |

## Accepted (no change required)

### Global Select tag styles in `global.css`

**Decision (team):** App-wide `.ant-select-selection-item-content` / tag SVG overrides in `global.css` are intentional and allowed.

**Original concern:** White tag text on all multi-selects app-wide.  
**Status:** Not pursuing a fix for SKSH-137.

## Positive notes

- Feature colocated under `src/pages/management/skillshow-users/` with split dashboard/onboarding, hooks, constants, and types
- Export handler uses ref on team users (avoids parent re-render churn)
- `cn()` used on dashboard cards; audit log uses `&&` conditional rendering
- Bulk update uses single `bulkPatchSkillshowUsers` API call
- Legacy generic user management pages removed; management routes redirect old `/management/user/*` to account settings
- Partner delete flow mirrors team users (`usePartnerActionMenu.tsx` + confirm modal)
- Connect Social modal partner tab/search refactor included

## Coordination notes (not severities)

- Routes are permission-driven (`usePermissionRoutes` + `import.meta.glob`); ensure backend permissions expose component paths such as `management/skillshow-users/dashboard/index` and onboarding paths matching `breadcrumb.ts` (`/user-management/skillshow-users/...`).

## Out of scope (per review prompts)

- Ad-hoc gray/blue Tailwind on Ant components
- `!` important modifiers on Ant overrides
- Token migration to theme CSS variables
- Global Select tag styling in `global.css` (team-accepted)
