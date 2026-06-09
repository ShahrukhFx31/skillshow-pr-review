# Backend PR Review — skillshow (`SKSH-296`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-296-1`  
**Base:** `main...HEAD` @ `bb59d79`  
**Initial review:** 2026-06-09  
**Re-reviewed:** 2026-06-09 — fixes verified on branch (`bb59d79` `fix: improvement and fixes`)  
**Scope:** Centralized audit logging for app-user, crew-user, skillshow-user, and partner create/update/bulk flows; entity-scoped audit-log list APIs; removal of `modificationOn` / `modificationBy` across user domains (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 (0 Critical, 4 High) — **2 Fixed**, **2 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `src/utils/audit-log.utils.ts` | **Introduced in this PR** — in scope for SKSH-296. **Accepted** |
| `src/utils/list-row-repository.utils.ts` | **Modified** — removed unused `runTouchModificationByUserIds` after audit migration. **Accepted** (dead-code removal; no remaining consumers) |
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `mongo-change-stream.service.ts`, `change-stream.utils.ts` | **Not modified** |

Entity-scoped `GET /:id/audit-logs` endpoints return a fixed newest-100 slice (`AUDIT_LOG_LIST_LIMIT`) without `createListQuerySchema` pagination — **Accepted** for embedded detail-panel UX.

---

## GitHub comments (Open findings)

*None — all findings Fixed or Accepted.*

---

## Findings

---
Delete operations do not write audit entries

Risk Level: HIGH
File Path: src/services/crew-user.service.ts
Lines: 249-255

Description:
**Contract / completeness.** Create, form update, and bulk update events are recorded through `*-audit.service` → `auditLogService`, but delete paths (`deleteCrewUser`, `deleteSkillshowUser`, `deleteAppUser`, `deletePartner`) do not append audit rows.

Impact:
- Deleted users/partners leave no centralized audit record of who performed the deletion or when.

Recommendation:
Add `recordDeleted` (or created-style marker) on all four delete service methods while `actorUserId` is available.

**PR comment (line 249):** **High:** Delete paths soft-delete without an audit entry — add delete audit marker in a follow-up.

**Status:** Accepted — deferred out of SKSH-296 scope; follow-up ticket.
---

---
Audit persistence runs after DB commit without transaction

Risk Level: HIGH
File Path: src/services/crew-user.service.ts
Lines: 184-196

Description:
**Reliability.** Patch flows persist entity changes then `await` audit recording with no Mongo transaction or outbox. Audit insert failure returns 500 after mutation is committed.

Impact:
- Data changed but no audit row; retries may skip diff on second attempt.

Recommendation:
Mongo session/transaction or durable async audit path with retry.

**PR comment (line 190):** **High:** Audit rows written after save with no transaction — accepted for v1.

**Status:** Accepted — intentional for v1; post-commit audit writes without transaction/outbox.
---

---
Duplicate reporting-manager snapshot logic across crew and skillshow audit services

Risk Level: HIGH
File Path: src/services/crew-user-audit.service.ts
Lines: 49-56

Description:
**DRY.** `crew-user-audit.service.ts` and `skillshow-user-audit.service.ts` duplicated reporting-manager snapshot pipeline.

Impact:
- Drift risk on manager-resolution fixes.

Recommendation:
Extract shared helper parameterized by field name + name-map resolver.

**PR comment (line 50):** **High (DRY):** Duplicate reporting-manager snapshot logic — extract shared helper.

**Re-review (`bb59d79`):** **Fixed** — shared `src/utils/audit-log-reporting-manager.utils.ts` (`buildReportingManagerAuditSnapshotPair`, `resolveReportingManagerNameMap`); both audit services delegate to it.
---

---
App-user retained modificationOn/By while crew/skillshow migrated

Risk Level: HIGH
File Path: src/services/app-user.service.ts
Lines: 454-528 (initial)

Description:
**Global consistency.** Crew/skillshow dropped embedded `modificationOn` / `modificationBy`; app-user initially kept dual-tracking alongside audit logs.

Impact:
- Two sources of “who last changed this” for app-users vs one for crew/skillshow.

Recommendation:
Remove app-user `modificationOn` / `modificationBy` writes and list projections; rely on audit logs only.

**PR comment (line 473):** **High (Global consistency):** App-user still writes modification metadata alongside audit logs.

**Re-review (`bb59d79`):** **Fixed** — `modificationOn` / `modificationBy` removed from `app-user.model.ts`, `app-user.repository.ts`, `app-user.service.ts`, and `app-user.types.ts`; audit-only path matches crew/skillshow.
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Delete operations do not write audit entries | HIGH | Accepted | src/services/crew-user.service.ts | 249-255 | 249 |
| 2 | Audit persistence runs after DB commit without transaction | HIGH | Accepted | src/services/crew-user.service.ts | 184-196 | 190 |
| 3 | Duplicate reporting-manager snapshot logic (crew vs skillshow) | HIGH | ✅ Fixed | src/utils/audit-log-reporting-manager.utils.ts | 36-66 | 50 |
| 4 | App-user retained modificationOn/By; crew/skillshow migrated | HIGH | ✅ Fixed | src/services/app-user.service.ts | — | 473 |

### Re-review notes (2026-06-09)

| Change | Verdict |
|--------|---------|
| `audit-log-reporting-manager.utils.ts` + thin crew/skillshow audit services | Fixes #3 DRY |
| App-user model/repo/service drop `modificationOn`/`By` | Fixes #4 global consistency |
| `list-row-repository.utils.ts` removes `runTouchModificationByUserIds` | Accepted dead-code cleanup |

### Positive notes

- Clean layering: `*-audit.service` → `auditLogService` → `auditLogRepository` → `audit-log.utils`.
- Shared reporting-manager snapshot util avoids crew/skillshow drift.
- Per-entity audit field allow-lists in `src/constants/*.constants.ts`.
- Entity audit routes: admin RBAC, param validation, registered before `/:id` catch-alls.
- Crew/skillshow bulk patches re-fetch list rows before diffing.
- Unit tests for `audit-log.utils`; domain tests mock audit services.

**Merge readiness:** **No open Critical/High blockers on the backend diff.**
