# Frontend PR Review — skillshow-admin-ui (`SKSH-440`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/366  
**Branch:** `SKSH-440` → main  
**Head:** `cf5b1490fd2346f74c17f7a7344a11724c5aa508`  
**Scope:** Partner `referralLink` field (form + connect flow), `ShareUrlCopyField` reuse, partners table inline status toggle  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/Completed/SKSH-440/backend.md` (skillshow #255)  
**Updated:** 2026-07-30 — re-review on latest head (all findings resolved)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

---
Status toggle mutation lacks error handling and directory cache invalidation

Risk Level: MEDIUM
File Path: src/pages/partners/dashboard/components/partners-table.tsx
Lines: 41-57

Description:
The inline `Switch` status mutation previously lacked `onError` and directory cache invalidation.

Impact:
- Resolved — latest head adds `toast.error` on failure and invalidates `partnersDirectoryQueryKey` on success.

Recommendation:
N/A — fixed on latest head.
---

---
Create mutation omits partnersDirectoryQueryKey invalidation

Risk Level: MEDIUM
File Path: src/pages/partners/onboarding/components/partner-form.tsx
Lines: 143-146

Description:
**Global consistency.** `patchPartnerMutation.onSuccess` invalidates `partnersDirectoryQueryKey`, but `createPartnerMutation.onSuccess` previously only invalidated `["partners", "list"]`.

Impact:
- Resolved — latest head adds `void queryClient.invalidateQueries({ queryKey: partnersDirectoryQueryKey });` to `createPartnerMutation.onSuccess` (line 145).

Recommendation:
N/A — fixed on latest head.
---

---
Referral link client validation weaker than backend http/https rule

Risk Level: MEDIUM
File Path: src/pages/partners/onboarding/constants.ts
Lines: 93-98

Description:
Referral Link field previously used Ant Design `{ type: "url" }`, accepting schemes the backend rejects.

Impact:
- Resolved — `partnerReferralLinkRules()` + `isHttpOrHttpsUrl` align client validation with backend http/https + max 2048.

Recommendation:
N/A — fixed on latest head.
---

**Positive notes:** `referralLink` flows through `PartnerFields` / list rows; audit labels auto-pick up via `buildAuditFieldLabelMap(FORM_SECTIONS)`; `ShareUrlCopyField` extension is backward-compatible; `showPartnerConnectInfo` opens validated API URLs with `noopener,noreferrer`; responsive table correctly reuses `partnerColumns` for the status switch; create/patch/status mutations all invalidate `partnersDirectoryQueryKey`.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Status toggle mutation lacks error handling and directory cache invalidation | MEDIUM | ✅ Fixed | src/pages/partners/dashboard/components/partners-table.tsx | 41-57 |
| 2 | Create mutation omits partnersDirectoryQueryKey invalidation | MEDIUM | ✅ Fixed | src/pages/partners/onboarding/components/partner-form.tsx | 143-146 |
| 3 | Referral link client validation weaker than backend http/https rule | MEDIUM | ✅ Fixed | src/pages/partners/onboarding/constants.ts | 93-98 |

**Merge readiness:** No open Critical/High/Medium blockers — approve for merge.
