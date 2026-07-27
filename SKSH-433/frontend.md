# PR review — skillshow-admin-ui #362 (SKSH-433)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-433 → main  
**Head:** `10056348c8b520433c3ecc08999335d2547e74e3`  
**Scope:** Open edit modal from My Videos list (nav state `openEdit`); actions cell → dropdown with View/Edit/Request edits/Delete  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

### `src/pages/videos/my-videos/dashboard/components/my-videos-table.tsx`

- **L19** — Responsive/mobile list path missing Edit (`canEditVideo` / `onEdit`)

### `src/pages/videos/details/index.tsx`

- **L162** — `openEdit` left in history state after consume (refresh reopens modal)

## Findings

---
Responsive/mobile list path missing Edit

Risk Level: HIGH
File Path: src/pages/videos/my-videos/dashboard/components/my-videos-table.tsx
Lines: 146-156

Description:
**Global consistency.** Desktop columns get `canEditVideo` / `onEdit` and expose Edit in the actions dropdown. The same `MyVideosTable` still renders `MyVideosTableResponsive` → `MobileVideoCard` **without** those props. `MyVideosTableResponsiveProps` / `MobileVideoCard` were not extended in this PR.

Impact:
- Mobile/narrow layout cannot open edit from the list (desktop-only feature in a shared list surface).

Recommendation:
Thread `canEditVideo` + `onEdit` through `MyVideosTableResponsive` and `MobileVideoCard` (or reuse the same dropdown/actions pattern). Update `MyVideosTableResponsiveProps` accordingly.
---

---
openEdit left in history state after consume

Risk Level: HIGH
File Path: src/pages/videos/details/index.tsx
Lines: 162-167

Description:
**Contract / UX.** Effect opens the edit modal once per mount via `openEditFromNavigationConsumed`, but never clears `openEdit` from `location.state`. A full remount (refresh, remounting the route) resets the ref while history still has `openEdit: true`, so the modal opens again.

Impact:
- Refresh / remount on a details URL navigated with Edit re-opens the edit modal unexpectedly.

Recommendation:
After consuming the flag, `navigate`/`setSearchParams` with `replace: true` and state that omits `openEdit` (preserve `linkedAthleteRelationId` / breadcrumb). Example:

```ts
openEditFromNavigationConsumed.current = true;
setEditOpen(true);
const { openEdit: _ignored, ...rest } = (location.state ?? {}) as Record<string, unknown>;
void navigate(`${location.pathname}${location.search}`, { replace: true, state: rest });
```
---

**Positive notes:** `canEditVideo` gates Edit + `onEdit`; `readVideoDetailsLocationState` extended cleanly; action keys centralized in `MY_VIDEOS_ACTION_KEYS`; delete still goes through `DestructiveActionConfirmModal`.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Responsive/mobile list path missing Edit | HIGH | Open | src/pages/videos/my-videos/dashboard/components/my-videos-table.tsx | 146-156 |
| 2 | openEdit left in history state after consume | HIGH | Open | src/pages/videos/details/index.tsx | 162-167 |

**Merge readiness:** Request changes — open High findings #1–#2.
