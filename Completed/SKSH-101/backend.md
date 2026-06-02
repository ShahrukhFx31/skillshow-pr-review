# Backend PR Review — skillshow (`SKSH-101`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-101`  
**Base:** `main...HEAD`  
**Re-verified:** `c39df4e` (`fix`) + merge `c941603`  
**Scope:** Layer separation, MongoDB/query performance, validation/security, types/constants, error handling (Critical & High only)  
**Findings:** 3 total — **2 fixed**, **1 accepted**, **0 open**

---

## Verification summary

| # | Title | Risk | Status | Evidence |
|---|--------|------|--------|----------|
| 1 | Live operations cached 30 min with real-time UX | HIGH | **Fixed** | Charts cached under `SUPER_ADMIN_CHARTS_CACHE_NAMESPACE` only; `buildKpis()` + `buildOperations()` run every request (`super-admin.controller.ts` 63–66) |
| 2 | Placeholder crew avg rating exposed as real metric | HIGH | **Fixed** | `avgRating` removed from `SuperAdminResponse` (`super-admin.types.ts` 34–38); crew rows map id/editor/activeAssignments only (`super-admin.service.ts` 201–206) |
| 3 | KPIs/ops rebuilt on every chart-range request (monolithic endpoint) | MEDIUM | **Accepted** | By product — keep monolithic handler; chart-only split deferred |

---

## Fixed (verified)

### #1 — Live operations no longer in long-lived full-response cache

**Was:** Entire `SuperAdminResponse` cached 30 minutes.

**Now:** `chartsCacheKey` uses `super-admin:charts` + date range; cache stores `SuperAdminChartsCached` only. Each `GET /dashboard` still returns fresh `kpis` and `operations`.

### #2 — Placeholder avg rating removed

**Was:** `SUPER_ADMIN_PLACEHOLDER_AVG_RATING` (4.8) on every crew row.

**Now:** DTO and service omit `avgRating`; repository layer extracted (`super-admin.repository.ts`).

---

## Accepted (no change required)

### #3 — KPIs/ops rebuilt on chart-range dashboard hits

**Decision (team):** Accept the monolithic `GET /dashboard` behavior—`buildKpis()` and `buildOperations()` on every request is acceptable; chart-only `?include=` / split route is out of scope for this PR.

**Evidence:** `super-admin.controller.ts` 63–66 — unchanged by design.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Live operations cached 30 min with real-time UX | HIGH | Fixed | `src/controllers/super-admin.controller.ts` | 27-73 |
| 2 | Placeholder crew avg rating exposed as real metric | HIGH | Fixed | `src/services/super-admin.service.ts` | 201-206 |
| 3 | KPIs/ops rebuilt on chart-range dashboard hits | MEDIUM | Accepted | `src/controllers/super-admin.controller.ts` | 63-66 |

**Positive notes (fix pass):** Repository extracted for super-admin reads; `flushCache` clears charts + legacy namespaces. `buildCharts` / `buildKpis` / `buildOperations` separation is clear.

**Skipped (per request):** Pagination/limit issues.
