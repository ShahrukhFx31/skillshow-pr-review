# Backend PR Review — skillshow (`SKSH-297`)

**Repo:** skillshow  
**Branch:** `SKSH-297`  
**Base:** `main...HEAD`  
**Scope:** Coach team tagging on videos (`teamId` / `teamName`), coach-only PATCH enforcement, list/detail response enrichment (Critical & High only)  
**Findings:** 0 Critical, 0 High

---

## Overview

The PR moves coach `teamId` PATCH handling out of `VideoController` into `videoService.applyTeamIdFromPatch`, enforces coach RBAC (403 for non-coach callers that include `teamId`), validates ownership via `teamRepository.findActiveIdForCoachLean`, and enriches GET/list/PATCH responses with `teamName` through batched `findNamesByIdsLean`. Linked-athlete PATCH reuses the same service path. Joi, Swagger, and unit tests are updated accordingly.

---

## Findings

No Critical or High issues identified in scope.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

### Positive notes

- Layer separation improved: controller delegates team validation/application to `videoService`; repository holds Mongo access only.
- List responses use `mapVideosToResponseWithTeam` with a single batched team-name lookup (no N+1 per row).
- `teamId` is validated in `updateVideoBodySchema`; coach-only mutation returns explicit 403 vs prior silent ignore.
- Soft-delete respected in `findNamesByIdsLean` (`isDeleted: { $ne: true }`).
- Controller and util tests cover team mapping and 403 on non-coach `teamId` PATCH.

**Merge readiness:** No open Critical/High blockers on the backend diff.
