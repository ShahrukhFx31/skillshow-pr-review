# PR review — skillshow #251 (SKSH-430)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-430 → main  
**Head:** `10b8fe6b0de882aa29007eaf61965aa42d214568`  
**Scope:** Admin teams CRUD/list under `/v1/admin/teams*`, reuse coach-link service, coach reassignment + notify  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** `pr-review/SKSH-430/frontend.md` (admin-ui #360)  
**Updated:** 2026-08-04 — re-verify on latest head (all prior High findings fixed)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

---
List handlers read req.query instead of validatedQuery

Risk Level: HIGH
File Path: src/controllers/admin-teams.controller.ts
Lines: 17-19, 196-198, 274-276

Description:
**Contract.** Controllers now use `getRequestQuery(req)`, which reads `validatedQuery` when Joi middleware ran.

Impact:
- Resolved — list endpoints honor coerced/bounded query params.

Recommendation:
N/A — fixed on head `10b8fe6b`.
---

---
Team/coach athlete list routes lack query validation

Risk Level: HIGH
File Path: src/routes/admin-teams.routes.ts
Lines: 41-46, 93-98

Description:
**Contract.** Athlete list routes now wire `validate(adminAthleteListQuerySchema, "query")`.

Impact:
- Resolved — invalid query shapes rejected at the edge with shared bounds/allow-lists.

Recommendation:
N/A — fixed on head `10b8fe6b`.
---

---
Admin list filter/count/find duplicate coach-scoped methods

Risk Level: HIGH
File Path: src/repositories/coach-link.repository.ts
Lines: 440-530

Description:
**DRY.** Admin and coach list paths share `teamListFilter`, `findTeamsListLean`, and thin wrappers; removed duplicate admin-only filter/find/copy and unused `findTeamListRowByIdLean`.

Impact:
- Resolved — single list implementation for coach + admin paths.

Recommendation:
N/A — fixed on head `10b8fe6b`.
---

---
countAllTeamsList bypasses soft-delete filter

Risk Level: HIGH
File Path: src/repositories/coach-link.repository.ts
Lines: 448, 530

Description:
**Contract.** `teamListFilter` always includes `isDeleted: false`; both `find` and `countDocuments` use it (coach + admin counts).

Impact:
- Resolved — pagination totals align with returnable rows after soft-deletes.

Recommendation:
N/A — fixed on head `10b8fe6b`.
---

**Positive notes:** `authorize({ roles: ["admin"] })` expands to `super_admin`. Admin mutations resolve the team’s coach and reuse coach-link eligibility. Coach role checked on create/reassign; roster trimmed on reassignment. Mandatory team logo + S3 cleanup on replace. `createdBy` tracking, notifications, and tests remain solid.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | List handlers read req.query instead of validatedQuery | HIGH | ✅ Fixed | src/controllers/admin-teams.controller.ts | 17-19, 196-198, 274-276 |
| 2 | Team/coach athlete list routes lack query validation | HIGH | ✅ Fixed | src/routes/admin-teams.routes.ts | 41-46, 93-98 |
| 3 | Admin list filter/count/find duplicate coach-scoped methods | HIGH | ✅ Fixed | src/repositories/coach-link.repository.ts | 440-530 |
| 4 | countAllTeamsList bypasses soft-delete filter | HIGH | ✅ Fixed | src/repositories/coach-link.repository.ts | 448, 530 |

**Merge readiness:** **Merge-ready** — ship with frontend #360.
