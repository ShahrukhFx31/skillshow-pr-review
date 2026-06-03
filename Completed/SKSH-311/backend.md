# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 — @ `05281a9` (`fix: bug fixes`)  
**Scope:** Server-driven pagination, search, sort, and filters for crew users, SkillShow users, and partners (Critical & High only)  
**Findings:** 2 (0 Critical, 0 High open) — **all prior findings fixed**

**Aligned with:** [frontend.md](./frontend.md)

### Re-verification verdict (2026-06-02)

| # | Original issue | Verdict |
|---|----------------|---------|
| 1 | `/directory` returned paginated admin list payload | **✅ Fixed** — dedicated `partnerController.directory` → `listPartnersDirectory()` → `listActiveDirectoryRows()` (flat `PartnerListRow[]`, active only) |
| 2 | List controllers fell back to raw `req.query` | **✅ Fixed** — `crew-user`, `partner`, and `skillshow-user` `list` use only `(req as ValidatedQueryRequest).validatedQuery` |

Additional fix in the same commit (supports frontend High #2): `GET /:partnerId` + `getPartner` / `findPartnerListRowByPartnerId`.

No new Critical/High issues identified in the post-fix diff.

---

---
`/partners/directory` returns paginated payload (breaks quick-connect consumers)

Risk Level: CRITICAL  
File Path: src/routes/partner.routes.ts  
Lines: 20-24

Description:
(Initial review.) `/directory` reused paginated `partnerController.list` with `partnerListQuerySchema`.

**Re-review:** ✅ **Fixed** — `/directory` calls `partnerController.directory` (no list query validation). `PartnerService.listPartnersDirectory()` returns a flat array from `listActiveDirectoryRows()`.

**PR comment (line 23):** **Critical:** Adding list query validation on `/directory` makes this route return `{ items, pagination }` … *(resolved — use separate handler; done.)*

---

---
List controllers merge raw `req.query` when `validatedQuery` is absent

Risk Level: HIGH  
File Path: src/controllers/crew-user.controller.ts  
Lines: 31-34

Description:
(Initial review.) `validatedQuery ?? req.query` on list handlers.

**Re-review:** ✅ **Fixed** — e.g. `...(req as ValidatedQueryRequest).validatedQuery` only (same in `partner.controller.ts` and `skillshow-user.controller.ts`).

**PR comment (line 33):** **High:** Please read list params only from `validatedQuery` … *(resolved.)*

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `/partners/directory` returns paginated payload | CRITICAL | ✅ Fixed | src/routes/partner.routes.ts | 20-24 |
| 2 | List controllers merge raw `req.query` fallback | HIGH | ✅ Fixed | src/controllers/crew-user.controller.ts | 31-34 |

## Positive notes

- Shared `createListQuerySchema`, `list-query-aggregation.utils`, and `DEFAULT_LIST_QUERY` reduce duplication across crew, team, and partner lists.
- Search uses `escapeRegex`; list queries cap `pageSize` at 100.
- Directory contract is explicit: active partners only, sorted, no pagination wrapper.
- `GET /partners/:partnerId` for detail loads without scanning the admin list.
- Team reporting managers exposed at `GET /skillshow-users/reporting-managers` (mirrors crew).

**Merge readiness:** Backend clear — no open Critical/High on `SKSH-311` @ `05281a9`. Archive with frontend when both reports are complete.
