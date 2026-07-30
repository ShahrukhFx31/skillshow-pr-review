# Frontend PR Review — skillshow-admin-ui (`SKSH-447`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/370  
**Branch:** `SKSH-447` → main  
**Head:** `4243a99b8de8e7fe799634564f82110a328d7867`  
**Scope:** `ProfileVisibilityBanner` on My Videos dashboard — public/private profile status, share URL display, copy link, settings CTA  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** _(none on branch `SKSH-447`)_  
**Updated:** 2026-07-30 — archived (#1 fixed on latest head)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

---
Private profile banner lacks settings navigation CTA

Risk Level: MEDIUM
File Path: src/pages/videos/my-videos/dashboard/components/profile-visibility-banner.tsx
Lines: 72-78

Description:
When the profile is private, the banner previously showed copy directing users to settings but rendered no action affordance.

Impact:
- Resolved — latest head adds **Go to Settings** `Link` to `ACCOUNT_SETTINGS_PATH` when `!isPublic`, mirroring the public **Copy Link** CTA slot.

Recommendation:
N/A — fixed on latest head.
---

**Positive notes:** Reuses `PROFILE_QUERY_KEYS.profileSettings` and `isProfilePublic` (cache-aligned with settings tab); `useCopyToClipboard` provides toast feedback; banner hidden until query succeeds; constants colocated in dashboard `constants.ts`; protected modules untouched.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Private profile banner lacks settings navigation CTA | Medium | ✅ Fixed | src/pages/videos/my-videos/dashboard/components/profile-visibility-banner.tsx | 72-78 |

**Merge readiness:** No open Critical/High/Medium blockers — approve for merge.
