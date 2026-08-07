# Frontend PR Review — skillshow-admin-ui (`SKSH-453`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/376  
**Branch:** `SKSH-453` → main  
**Head:** `915135cb61252d3bbf4a31fcf7cbc751ea161e51`  
**Scope:** Partner CSV import headers aligned with backend; sample row includes Host Code; status casing fix  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired:** `pr-review/SKSH-453/backend.md` (#263)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:**
- `PARTNER_IMPORT_CSV_HEADERS` matches backend `IMPORT_TOOL_PARTNER_HEADERS` column order and labels (excludes onboarding-only fields like Referral Link).
- `getPartnerImportCsvHeaders()` replaces form-derived headers — import sample/template now matches API import contract.
- Sample row adds `"Host Code": "ELITE11"` and fixes `Status` to lowercase `"active"` (backend `PARTNER_STATUSES`).
- Typed `PARTNER_SAMPLE_VALUES` against header const improves drift detection.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | — | — | — | — | — |

**Merge readiness:** **Merge-ready** — ship with backend #263.
