# Frontend PR Review — skillshow-admin-ui (`INSTALOGIN`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/372  
**Branch:** `feat/insta-login` → main  
**Head:** `7b707e31316f79c85ba97b44a6c8ab3301c2e134`  
**Scope:** Instagram Login prereq modal, reconnect UX for legacy Facebook-Login tokens, copy updates  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired:** `pr-review/INSTALOGIN/backend.md` (#260), `pr-review/INSTALOGIN/orchestrator.md` (#30)

## GitHub comments

### `src/pages/dashboard/components/ConnectSocialModal.tsx`

- **L164** — OAuth start API errors not caught (no toast)

### `src/pages/user/account/share-account-tab.tsx`

- **L88** — OAuth start API errors not caught (no toast)

## Findings

---
Upload/distribute still treats legacy Instagram as connected

Risk Level: HIGH
File Path: src/pages/videos/utils/platformConnections.ts
Lines: 45-63

Description:
**Global consistency / DRY.** This PR adds `needsReconnect` UX in account settings and `ConnectSocialModal`, but `getConnectedPlatforms` and `findSocialConnectionForPlatform` still treat any active connection with credentials as connected. Legacy Facebook-Login Instagram tokens (flagged `needsReconnect` by backend #260) remain selectable on **upload** and **distribute** flows (`distribute-eligibility.utils.ts`, `ConnectedPlatformSelector`, `DistributeModal`).

Impact:
- Users can select Instagram for distribution while holding expired/legacy tokens.
- Publish jobs fail at orchestrator runtime instead of being blocked in UI with “Reconnect required”.
- Inconsistent UX vs account/share-account surfaces that already show amber reconnect state.

Recommendation:
Centralize effective-connection checks, e.g. `isSocialConnectionUsable(c)` returning false when `needsReconnect === true` or `metadata.needsReconnect === true`. Use it in `getConnectedPlatforms`, `findSocialConnectionForPlatform`, and any distribute/upload eligibility helpers. Optionally surface Instagram in upload selector as disabled with reconnect CTA linking to account settings.
---

---
OAuth start API errors not caught in ConnectSocialModal

Risk Level: MEDIUM
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx
Lines: 164-186

Description:
`startOAuthConnect` wraps `connectMutation.mutateAsync` in `try/finally` without `catch`. Network/API failures throw before the popup opens, leaving the user with no feedback (only `connecting` spinner clearing).

Impact:
- Silent failure when `/v1/vendors/connect/...` errors (500, offline, validation).
- Poor UX compared to popup error handling already present for `result.status === "error"`.

Recommendation:
Add `catch (err) { toast.error(getErrorMessage(err) ?? "Failed to start connection"); }` before `finally`, matching patterns used elsewhere in the app.
---

---
OAuth start API errors not caught in share-account-tab

Risk Level: MEDIUM
File Path: src/pages/user/account/share-account-tab.tsx
Lines: 88-110

Description:
Same issue as `ConnectSocialModal`: `startOAuthConnect` lacks a `catch` around `connectSocialStart` / popup flow.

Impact:
- Silent failures on Instagram reconnect from account settings.

Recommendation:
Add `catch` with `toast.error` before `finally`, consistent with disconnect handlers on the same page.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Upload/distribute still treats legacy Instagram as connected | HIGH | Open | src/pages/videos/utils/platformConnections.ts | 45-63 |
| 2 | OAuth start API errors not caught in ConnectSocialModal | MEDIUM | Open | src/pages/dashboard/components/ConnectSocialModal.tsx | 164-186 |
| 3 | OAuth start API errors not caught in share-account-tab | MEDIUM | Open | src/pages/user/account/share-account-tab.tsx | 88-110 |

**Merge readiness:** Not merge-ready — wire `needsReconnect` into upload/distribute eligibility before merge; add OAuth start error toasts.

**Summary-only (line not in diff):** Finding #1 — `platformConnections.ts` unchanged in this PR; update that shared helper (or re-export from a new util touched here) so video flows honor reconnect state.
