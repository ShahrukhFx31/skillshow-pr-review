# PR review — skillshow-admin-ui #350 (SKSH-389)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-389 → main  
**Head:** `9cabdbe0bb5d867795f79fe057126318f3a149d0`  
**Scope:** Complimentary edit-request UX, My Videos “Request edits”, video library bulk upload UI  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

### `src/pages/editRequest/components/UploadVideosScreen.tsx` (line 75)

**MEDIUM** — Complimentary copy still says SkillShow; backend is library-only

## Findings

---
Complimentary UX copy still describes SkillShow deliveries

Risk Level: MEDIUM
File Path: src/pages/editRequest/components/UploadVideosScreen.tsx
Lines: 74-76

Description:
**Contract / Global consistency** with skillshow #244. Backend complimentary eligibility is now **video-library tags only** (`inVideoLibrary`, not `isSkillshowUploaded`). This screen (and related type comments / sublabels) still say “SkillShow deliveries” / “SkillShow delivery”, which conflicts with the server contract athletes will hit on create and with list eligibility flags.

Impact:
- Athletes may expect SkillShow Uploaded rows to qualify for free edits when they do not.
- Support/admin confusion when UI wording and API errors diverge.

Recommendation:
Align copy and JSDoc with library-only complimentary edits (e.g. “Video library deliveries…”). Keep `freeEditEligible` driven by the API; remove SkillShow-uploaded implications from free-edit helpers/state shaping if unused.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Complimentary UX copy still describes SkillShow deliveries | MEDIUM | Open | src/pages/editRequest/components/UploadVideosScreen.tsx | 74-76 |

**Merge readiness:** No Critical/High blockers on frontend; one **Medium** copy/contract mismatch with library-only free edits on #244.
