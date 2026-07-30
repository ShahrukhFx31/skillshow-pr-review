# Backend PR Review — skillshow (`SKSH-440`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/255  
**Branch:** `SKSH-440` → main  
**Head:** `6a0016311c99cca76ecc8e6b48d12b8eb1fcf279`  
**Scope:** Partner `referralLink` model field, validation, service patch/clear logic, audit + tests  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** `pr-review/Completed/SKSH-440/frontend.md` (admin-ui #366)  
**Updated:** 2026-07-30 — re-review on latest head (unchanged scope, no new findings)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:**
- `referralLink` added consistently to model, types, `LIST_ROW_PROJECT`, `PATCHABLE`, and `PARTNER_AUDIT_FIELDS`.
- **DRY:** `CLEARABLE_STRING_FIELDS` consolidates empty/null unset logic (includes `referralLink`).
- Validation restricts URLs to `http`/`https` with `max(2048)` and allows `""`/`null` for clearing.
- Tests cover create, get, directory list, patch clear, and validation rejections for non-http schemes.
- No route/auth changes; no protected-module edits; no security policy regressions.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** Approve for merge — no open Critical/High/Medium findings.
