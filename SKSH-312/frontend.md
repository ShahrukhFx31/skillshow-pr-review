# Frontend PR Review — skillshow-admin-ui (`SKSH-312`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-312` → **`SKSH-178-1`** (stacked PR; compare with `git diff SKSH-178-1...HEAD`, not `main`)  
**Scope:** User `seq` migration in app / crew / skillshow management UI (dashboard actions, columns, detail links, types) — Critical, High & Medium  
**Findings:** 1 (0 High, 1 Medium)

> **Scope note:** Pagination omitted per review request. App-users pages (`onboarding/index.tsx`, `appUserService.ts`, activity panel, etc.) are on `SKSH-178-1` and are **not** in this PR diff — findings there are out of scope for GitHub inline comments on this PR.

---

---
Navigation and route matching assume every row has a valid `seq`

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/dashboard/hooks/use-app-user-actions.tsx  
Lines: 34-43, 75  
File Path: src/pages/management/app-users/onboarding/utils.ts  
Lines: 171-174  
File Path: src/pages/management/app-users/dashboard/components/app-users-columns.tsx  
Lines: 44-54

Description:
This PR switches `userId` / `appUserId`-style keys to `record.seq` (`String(record.seq)` for routes, resend, delete, and links). `appUserDetailMatchesRoute` now only compares `String(detail.seq) === routeUserId` and no longer accepts `mongoUserId`. If the API returns `seq: null` (list projection allows it) or `seq: 0` (linked users from `resolveLinkedUserSeq` fallback), navigation goes to `/view/0` or `/view/null` and detail hydration can fail to match the route param.

Impact:
- Broken view/edit/activity links for users not yet backfilled with `seq`.
- Detail page stuck loading when route param cannot match `detail.seq`.

Recommendation:
Guard UI: disable row actions when `seq == null`; show ID column as `—` until assigned. Keep temporary `mongoUserId` fallback in `appUserDetailMatchesRoute` until backfill completes, or align with backend excluding null-`seq` rows from list.

**PR comment (`use-app-user-actions.tsx` line 34):**  
**Medium:** Actions now navigate with `String(record.seq)` only. Please guard when `seq` is null/0 (or ensure API backfill) so rows without a assigned sequence are not linked to invalid routes.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Navigation assumes valid `seq` on every row | MEDIUM | Open | src/pages/management/app-users/dashboard/hooks/use-app-user-actions.tsx | 34-43 |

**Positive notes:** App / crew / skillshow dashboards consistently use `seq` for view, edit, resend, delete, and bulk selection; linked-user table links updated in `app-user-detail-columns.tsx`; types aligned with API (`AppUserFields.seq`).

**Out of scope (on `SKSH-178-1`, not in this PR diff):** Activity double-fetch, activity summary caps, form `resetFields` race, full-list client fetch.

**Merge readiness:** No open High on frontend; 1 open Medium (pairs with backend `seq` backfill). Backend has 2 open High — see `backend.md`.
