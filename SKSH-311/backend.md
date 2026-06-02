# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Scope:** Server-driven pagination, search, sort, and filters for crew users, SkillShow users, and partners (Critical & High only)  
**Findings:** 2 (1 Critical, 1 High)

---

---
`/partners/directory` returns paginated payload (breaks quick-connect consumers)

Risk Level: CRITICAL  
File Path: src/routes/partner.routes.ts  
Lines: 23-24

Description:
`GET /v1/partners/directory` now adds `validate(partnerListQuerySchema, "query")` (line 23) and still uses `partnerController.list`, so the response is `PaginatedResponse<PartnerListRow>` (`{ items, pagination }`) with default `pageSize: 10`, not a flat `PartnerListRow[]`.

`skillshow-admin-ui` still calls this route via `listPartnersDirectory()`, which types the response as `PartnerRow[]` and treats the body as an array (`ConnectSocialModal` filters/maps `allPartners`).

Impact:
- Quick Connect partner tabs call `.filter` on a non-array → runtime failure or empty UI.
- Even with a fixed client unwrap, only the first page (default 10 partners) is available; most active partners never appear.

Recommendation:
Keep directory as a dedicated contract, e.g. a separate controller/service method that returns all active partners (with a sane cap) or a flat array, and do not reuse the paginated admin list for authenticated directory consumers. If pagination is required on directory, document it and update every caller to pass `pageSize` and unwrap `items`.

**PR comment (line 23):** **Critical:** Adding list query validation on `/directory` makes this route return `{ items, pagination }` (default `pageSize: 10`) via `partnerController.list`, but `listPartnersDirectory` still expects a full array—Quick Connect will break and only see the first page. Keep directory unpaginated or use a separate handler/flag.

---

---
List controllers merge raw `req.query` when `validatedQuery` is absent

Risk Level: HIGH  
File Path: src/controllers/crew-user.controller.ts  
Lines: 31-35

Description:
`list` builds the query with `...(((req as ValidatedQueryRequest).validatedQuery ?? req.query) as Record<string, unknown>)`. The same pattern is used in `partner.controller.ts` and `skillshow-user.controller.ts`.

Routes wire `validate(..., "query")`, so `validatedQuery` should always be present on these paths. Falling back to `req.query` bypasses Joi coercion (strings for `page`/`pageSize`, unvalidated `sortBy`) if middleware is removed or mis-ordered.

Impact:
- Violates the project rule to trust only `validatedQuery` for list params.
- Accidental route misconfiguration can cause wrong pagination math or Mongo errors instead of a 400 from Joi.

Recommendation:
Use only `validatedQuery` after a narrow type guard, e.g. `const query = { ...DEFAULT_LIST_QUERY, ...(req as ValidatedQueryRequest).validatedQuery } as CrewUserListQuery`, and drop the `req.query` fallback.

**PR comment (line 33):** **High:** Please read list params only from `validatedQuery` (no `req.query` fallback) so pagination/sort stay Joi-coerced and invalid input fails at the middleware layer.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `/partners/directory` returns paginated payload | CRITICAL | Open | src/routes/partner.routes.ts | 23-24 |
| 2 | List controllers merge raw `req.query` fallback | HIGH | Open | src/controllers/crew-user.controller.ts | 31-35 |

## Positive notes

- Shared `createListQuerySchema`, `list-query-aggregation.utils`, and `DEFAULT_LIST_QUERY` reduce duplication across crew, team, and partner lists.
- Search uses `escapeRegex`; list queries cap `pageSize` at 100.
- Partners use `countDocuments` for totals; user lists use parallel aggregate + `$count` after lookups.
- Joi validation added on list routes; services return consistent `PaginatedResponse`.

**Merge readiness:** Blocked until directory contract is fixed (Critical #1). Address High #2 before merge or track as a fast follow-up.
