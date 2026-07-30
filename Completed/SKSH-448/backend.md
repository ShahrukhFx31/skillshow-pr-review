# Backend PR Review — skillshow (`SKSH-448`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/258  
**Branch:** `SKSH-448` → main  
**Head:** `c6063f9084da38c6b04eca24e0c49604f824292a`  
**Scope:** Admin bypass for SkillShow-sourced video metadata guards — athletes remain visibility-only; admins can manage Video Library uploads  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** _(none on branch `SKSH-448`)_  
**Related:** Builds on SKSH-445 SkillShow edit restrictions (`pr-review/Completed/SKSH-445/`)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:**
- **Security / RBAC:** Centralized `assertAthleteSkillshowEditAllowed` gates SkillShow metadata edits; `RBACService.hasAnyRole(userId, ["admin"])` exempts admins (includes `super_admin` via `expandAdminRoleRequirement`). Athletes/coaches/parents still hit `assertSkillshowAthleteEditAllowed`.
- **Global consistency:** All prior direct `assertSkillshowAthleteEditAllowed` call sites in `VideoService` migrated; controllers/services pass `userId` / `actorUserId` through async guards.
- **Contract:** Upload thumbnail presigned + record paths keep `respondIfAppError` for 403 surfacing; tests cover athlete block + admin allow on library thumbnails and admin metadata PATCH on library rows.
- Protected modules untouched; Video Library admin routes remain separately gated by `authorize({ roles: ["admin"] })`.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** Approve for merge — no open Critical/High/Medium findings.
