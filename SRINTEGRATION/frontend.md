# Frontend PR Review — skillshow-admin-ui (`SRINTEGRATION`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/371  
**Branch:** `feat/srintegration` → main  
**Head:** `f757e51df3df66bb6867625878c9067d6df36a7b`  
**Scope:** SportsRecruits connect UI — redirect + poll flow, platform constants, share-account tab, distribute boundaries  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired:** `pr-review/SRINTEGRATION/backend.md` (#259), `pr-review/SRINTEGRATION/orchestrator.md` (#29)

## Developer replies

No developer replies on open threads. Commit `f757e51` addresses all prior findings directly.

## GitHub comments

_(none — all prior Open findings resolved)_

## Findings

---
ConnectSocialModal SR poll reads stale cache after invalidate

Risk Level: HIGH
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx
Lines: 186-196

Description:
**Global consistency / Contract.** SportsRecruits connect poll calls `invalidateQueries` then immediately `getQueryData` without `refetch()` or `fetchQuery`. Invalidation does not refresh cache synchronously, so the linked check often runs on stale data. `share-account-tab.tsx` correctly uses `await refetch()` in its poll loop.

Impact:
- Dashboard connect modal may never detect successful SR linking; user sees silent failure/timeouts despite webhook success.

Recommendation:
Match share-account: `const { data: latest } = await refetch()` inside the poll callback, or `await queryClient.fetchQuery({ queryKey: vendorSocialConnectionsQueryKey })`.

**Re-verify (f757e51):** ✅ Fixed — both surfaces use shared `openApiVendorLinkAndPoll` with network `fetchConnections` (`refetch()`).
---

---
Silent failure paths after SR connect attempt

Risk Level: HIGH
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx
Lines: 200-216

Description:
On poll timeout, popup closed, or stale-cache miss there is no `toast.error` and no distinction from success except missing success toast.

Impact:
- Users cannot tell whether linking failed, is still pending, or succeeded without UI refresh.

Recommendation:
Add error toasts for timeout/abort/blocked popup; mirror social OAuth error handling.

**Re-verify (f757e51):** ✅ Fixed — `apiVendorLinkErrorMessage` toasts for timeout/aborted/popup_blocked.
---

---
Share-account connect start lacks catch / user-visible errors

Risk Level: HIGH
File Path: src/pages/user/account/share-account-tab.tsx
Lines: 99-132

Description:
`handleConnect` uses try/finally without catch for `connectApiStart` / `connectSocialStart` failures.

Impact:
- Network/API errors fail silently on the share-account path.

Recommendation:
Add catch with `toast.error` (and optional logging).

**Re-verify (f757e51):** ✅ Fixed — `catch` with `getApiErrorMessage` + toast on both paths.
---

---
ConnectApiVendorModal and connectApiVendor are unused dead code

Risk Level: MEDIUM
File Path: src/pages/components/ConnectApiVendorModal.tsx
Lines: 1-54

Description:
**KISS.** Modal and POST `connectApiVendor` are added but nothing imports or renders the modal. Runtime path is redirect-only via `connectApiStart`.

Recommendation:
Wire as admin/fallback path or remove until needed to avoid drift with backend `POST /connect/api` policy.

**Re-verify (f757e51):** ✅ Fixed — component deleted; redirect-only flow via `connectApiStart`.
---

---
Duplicate SR redirect+poll logic across modal and share-account tab

Risk Level: MEDIUM
File Path: src/utils/api-vendor-link-popup.ts
Lines: 1-80

Description:
**DRY.** ~35 lines duplicated with share-account tab; divergence already caused the cache bug in the modal only.

Recommendation:
Extract shared hook/helper, e.g. `connectRedirectVendorWithPoll({ vendorName, onLinked })`.

**Re-verify (f757e51):** ✅ Fixed — `openApiVendorLinkAndPoll` + `apiVendorLinkErrorMessage` shared util with explicit stale-cache warning in docstring.
---

**Positive notes:** `ALL_KNOWN_PLATFORMS` intentionally omits SportsRecruits (distribute-only separation); API connections treated alongside social in list helpers; feature flags via `useSocialPlatformAvailability`; `pollUntil` utility reused.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | ConnectSocialModal SR poll reads stale cache after invalidate | HIGH | ✅ Fixed | src/utils/api-vendor-link-popup.ts | 28-58 |
| 2 | Silent failure paths after SR connect attempt | HIGH | ✅ Fixed | src/pages/dashboard/components/ConnectSocialModal.tsx | — |
| 3 | Share-account connect start lacks catch / user-visible errors | HIGH | ✅ Fixed | src/pages/user/account/share-account-tab.tsx | — |
| 4 | ConnectApiVendorModal and connectApiVendor are unused dead code | MEDIUM | ✅ Fixed | _(removed)_ | — |
| 5 | Duplicate SR redirect+poll logic across modal and share-account tab | MEDIUM | ✅ Fixed | src/utils/api-vendor-link-popup.ts | 1-80 |

**Merge readiness:** **Merge-ready** — all findings resolved. Ship coordinated with backend #259 and orchestrator #29.
