# Backend PR Review — skillshow (`SKSH-297`)

**Repo:** skillshow  
**Branch:** `SKSH-297`  
**Base:** `main...HEAD`  
**Initial review:** 2026-06-03  
**Re-reviewed:** 2026-06-03 — no backend changes since initial review (`831baf7` + merge `657e782`)  
**Scope:** Coach team tagging on videos (`teamId` / `teamName`), coach-only PATCH enforcement, list/detail response enrichment (Critical & High only)  
**Findings:** 0 Critical, 0 High

---

## Overview

The PR moves coach `teamId` PATCH handling into `videoService.applyTeamIdFromPatch`, enforces coach RBAC (403 for non-coach callers that include `teamId`), validates ownership via `teamRepository.findActiveIdForCoachLean`, and enriches GET/list/PATCH responses with `teamName` through batched `findNamesByIdsLean`. Linked-athlete PATCH reuses the same service path.

---

## Findings

No Critical or High issues identified in scope.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

### Re-review notes (2026-06-03)

No new backend commits addressing review feedback were required; frontend fix `01e554e8` aligns with existing coach-only `teamId` API behavior.

### Positive notes

- Layer separation: controller → `videoService` → repository.
- Batched `teamName` enrichment on list responses.
- Joi validation and explicit 403 for non-coach `teamId` PATCH.
- Tests cover team mapping and 403 behavior.

**Merge readiness:** No open Critical/High blockers on the backend diff.
