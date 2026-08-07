# Frontend PR Review — skillshow-admin-ui (`INSTALOGIN`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/372  
**Branch:** `feat/insta-login` → main  
**Head:** `9a5961fb29cabbf35083487707667ce61ffe3da3`  
**Scope:** Instagram Login prereq modal, reconnect UX, platform connection eligibility for upload/distribute  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired:** `pr-review/Completed/INSTALOGIN/backend.md` (#260), `pr-review/Completed/INSTALOGIN/orchestrator.md` (#30)  
**Updated:** 2026-08-07 — archived (merge-ready)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

---
Upload/distribute still treats legacy Instagram as connected

Risk Level: HIGH
File Path: src/pages/videos/utils/platformConnections.ts
Lines: 44-78

Description:
**Global consistency / DRY.** Added `connectionNeedsReconnect`, `isSocialConnectionPresent`, and `isSocialConnectionUsable`; `getConnectedPlatforms` and `findSocialConnectionForPlatform` now exclude `needsReconnect` connections.

Impact:
- Resolved — upload/distribute flows honor reconnect state via shared helpers.

Recommendation:
N/A — fixed on head `9a5961fb`.
---

---
OAuth start API errors not caught in ConnectSocialModal

Risk Level: MEDIUM
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx
Lines: 223-224

Description:
`startOAuthConnect` now catches API/popup failures and shows `toast.error` via `getApiErrorMessage`.

Impact:
- Resolved — users get feedback when connect start fails.

Recommendation:
N/A — fixed on head `9a5961fb`.
---

---
OAuth start API errors not caught in share-account-tab

Risk Level: MEDIUM
File Path: src/pages/user/account/share-account-tab.tsx
Lines: 134-135

Description:
Same `catch` + `toast.error` pattern added to `startOAuthConnect`.

Impact:
- Resolved — account-settings reconnect surfaces errors.

Recommendation:
N/A — fixed on head `9a5961fb`.
---

**Positive notes:** `InstagramConnectPrereqModal`, reconnect UX in `SocialAccountsSection`, and API vendor link polling in connect flows remain coherent.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Upload/distribute still treats legacy Instagram as connected | HIGH | ✅ Fixed | src/pages/videos/utils/platformConnections.ts | 44-78 |
| 2 | OAuth start API errors not caught in ConnectSocialModal | MEDIUM | ✅ Fixed | src/pages/dashboard/components/ConnectSocialModal.tsx | 223-224 |
| 3 | OAuth start API errors not caught in share-account-tab | MEDIUM | ✅ Fixed | src/pages/user/account/share-account-tab.tsx | 134-135 |

**Merge readiness:** **Merge-ready** — review archived.
