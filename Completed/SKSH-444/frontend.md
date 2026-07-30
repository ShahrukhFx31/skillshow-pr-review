# Frontend PR Review — skillshow-admin-ui (`SKSH-444`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/368  
**Branch:** `SKSH-444` → main  
**Head:** `2951884d17bdcd32a0d6441315b26a8c70d6c3b9`  
**Scope:** Fix email notification pref persistence (immediate PATCH + cache seed); inline video visibility toggle on My Videos list  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-444/backend.md` (skillshow #257)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:**
- **Email prefs:** Removes debounced batch PATCH; saves one key per toggle with optimistic React Query cache writes, cache seed on mount, and `forceMount` on Settings tab to prevent remount flash. `onSuccess` merges patched keys over API response to avoid stale snap-back.
- **Video visibility:** `VideoVisibilityToggle` reuses `VideoVisibilityCue` when read-only; list/mobile/desktop columns wired consistently. Mutation sends `{ isPublic }` only via `patchVideoForContext` — aligned with SkillShow visibility-only backend rules.
- Optimistic list update + rollback on error; per-row loading state; `canEditVideo` gates interactivity.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** Approve for merge — no open Critical/High/Medium findings.
