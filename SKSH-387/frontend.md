# PR review — skillshow-admin-ui #349 (SKSH-387)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-387 → main  
**Scope:** Super Admin dashboard layout, KPI order, live-ops table links  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

### `src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx` (line 93)

**HIGH** — Dashboard links require API fields the backend does not return yet

## Findings

---
Dashboard drill-down links missing API contract fields

Risk Level: HIGH
File Path: src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx
Lines: 93-137

Description:
**Contract / Global consistency.** Column renderers link crew editors via `record.seq` (`crewUserRoutes.view`) and events via `record.slug` (`buildEventViewPath`). Types and DTOs were extended with `seq` and `slug`, and `activeCrew` maps `seq` from the API, but `upcomingEvents` is still passed through unchanged. On current `skillshow` `main`, `SuperAdminService.buildOperations()` returns `activeCrew` without `seq` and `upcomingEvents` without `slug` (only `id`, `event`, dates, `status`). Until the dashboard API is updated in the same release, links never render (both renderers fall back to plain text).

Impact:
- Super Admin “Active Events” and crew editor cells do not navigate as intended after this UI ships alone.
- Frontend/backend DTO drift (`dashboard.types.ts` vs actual JSON).

Recommendation:
Ship a matching backend change (or same ticket) to include `slug` on operational events (from `EventModel`) and crew `seq` (lookup by editor `id`) in the super-admin dashboard payload, then map `slug` in `useSuperAdminDashboard` the same way as `seq`. Cross-check `pr-review/SKSH-387/backend.md` if added. Until then, document as blocked on API or defer link renderers.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dashboard drill-down links missing API contract fields | HIGH | Open | src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx | 93-137 |

**Merge readiness:** Blocked on open **High** — align super-admin API with `seq` / `slug` or scope link UI to a follow-up with API.
