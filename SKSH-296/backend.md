# Backend PR Review — skillshow (`SKSH-296`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-296-1`  
**Base:** `main...HEAD` @ `c4b7573`  
**Initial review:** 2026-06-09  
**Scope:** Centralized audit logging for app-user, crew-user, skillshow-user, and partner create/update/bulk flows; entity-scoped audit-log list APIs; removal of `modificationOn` / `modificationBy` from crew/skillshow models and list projections (Critical & High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 (0 Critical, 4 High) — **4 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `src/utils/audit-log.utils.ts` | **Introduced in this PR** — in scope for SKSH-296 (audit-log ticket). **Accepted** as intentional establishment of the frozen module. |
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `change-stream.utils.ts` | **Not modified** |

Entity-scoped `GET /:id/audit-logs` endpoints return a fixed newest-100 slice (`AUDIT_LOG_LIST_LIMIT`) without `createListQuerySchema` pagination — **Accepted** for embedded detail-panel UX; not a global admin audit table.

---

## GitHub comments (Open findings)

*None — all backend findings Accepted.*

---

## Findings

---
Delete operations do not write audit entries

Risk Level: HIGH
File Path: src/services/crew-user.service.ts
Lines: 249-255

Description:
**Contract / completeness.** All four domain services now record create, form update, and bulk update events through `*-audit.service` → `auditLogService`, but none of the delete paths (`deleteCrewUser`, `deleteSkillshowUser`, `deleteAppUser`, `deletePartner`) append an audit row. Soft-delete is a primary admin action that compliance users expect in an audit trail.

Impact:
- Deleted users/partners leave no centralized audit record of who performed the deletion or when.
- Audit panel on view pages cannot show delete history; operators must infer from absence of the entity.

Recommendation:
Add `auditLogService.recordCreated` (or a dedicated `recordDeleted` with message `"User deleted"` / `"Partner deleted"`) in each delete service method while `actorUserId` is available from the controller:

```typescript
await crewUserAuditService.recordDeleted({
  entityId: seqParam,
  subjectUserId: user._id,
  updatedBy: actorUserId,
});
```

Wire `recordDeleted` once in `AuditLogService` and delegate from each `*-audit.service`.

**PR comment (line 249):** **High:** `deleteCrewUser` (and sibling delete methods on app-user, skillshow-user, partner) soft-delete without an audit entry. Create/update/bulk are covered — please add a delete audit marker on all four delete flows while `actorUserId` is still available from the controller.

**Status:** Accepted — deferred out of SKSH-296 scope; delete audit entries to be addressed in a follow-up ticket.
---

---
Audit persistence runs after DB commit without transaction

Risk Level: HIGH
File Path: src/services/crew-user.service.ts
Lines: 184-196

Description:
**Reliability / Contract.** Patch flows persist entity changes (`user.save()`, `crewUser.save()`, repository `updateByPartnerId`, etc.) and only then `await` audit recording. There is no Mongo transaction or outbox. If `auditLogRepository.create` throws, the handler surfaces 500 to the client while the mutation is already committed.

Impact:
- Data changed but no audit row — exactly the gap this ticket is meant to close.
- Client may retry; second attempt may diff identical snapshots and skip audit (`changes.length === 0`), worsening the gap.

Recommendation:
Use a Mongo session/transaction wrapping the domain update and `auditLogRepository.create`, or move audit writes to a durable async path with retry (change-stream handler is out of scope here). At minimum, log and alert on audit write failure without falsely implying the whole operation failed if business data already saved.

**PR comment (line 190):** **High:** Audit rows are written after `user.save()` / `crewUser.save()` with no transaction. If `auditLogRepository.create` fails, the API returns 500 but the profile change is already persisted. Consider a Mongo transaction or durable retry so updates and audit trails stay consistent.

**Status:** Accepted — intentional for v1; post-commit audit writes accepted without Mongo transaction/outbox in this ticket.
---

---
Duplicate reporting-manager snapshot logic across crew and skillshow audit services

Risk Level: HIGH
File Path: src/services/crew-user-audit.service.ts
Lines: 99-152

Description:
**DRY.** `crew-user-audit.service.ts` and `skillshow-user-audit.service.ts` are ~90% identical: `buildSnapshotPair`, `applyReportingManagerNames`, `collectReportingManagerIds`, and `resolveReportingManagerNameMap` differ only in the manager ID format (ObjectId string vs user seq) and the repository lookup (`findDisplayNameMapByIds` vs `findDisplayNameMapBySeqs`).

Impact:
- Future field-label or manager-resolution fixes must be applied twice and will drift.
- Increases review and test surface for every audit enhancement.

Recommendation:
Extract shared helpers under `src/utils/audit-log.utils.ts` (consumer-only additions) or a small `reporting-manager-audit.utils.ts`:

```typescript
export async function buildManagerLabeledSnapshots(
  oldRow: Record<string, unknown> | undefined,
  newRow: Record<string, unknown>,
  fields: readonly string[],
  managerField: string,
  resolveNameMap: (rows: Record<string, unknown>[]) => Promise<Map<string, string>>,
): Promise<[Record<string, string | null>, Record<string, string | null>]> { /* ... */ }
```

Keep entity-specific audit services thin: constants + `recordCreated` / `list` / field lists only.

**PR comment (line 99):** **High (DRY):** `CrewUserAuditService` and `SkillshowUserAuditService` duplicate the reporting-manager snapshot pipeline (`buildSnapshotPair`, name-map resolution). Extract a shared helper parameterized by field name + resolver (`findDisplayNameMapByIds` vs `findDisplayNameMapBySeqs`) so the two services don't drift.

**Status:** Accepted — duplication accepted for v1; shared extraction deferred to a follow-up refactor.
---

---
App-user still uses embedded modification metadata; crew/skillshow migrated in same PR

Risk Level: HIGH
File Path: src/services/app-user.service.ts
Lines: 367-368, 400-401, 454-456, 502-505, 525-528

Description:
**Global consistency.** Crew and skillshow users drop `modificationOn` / `modificationBy` from models, repositories, and list projections and rely solely on the new `AuditLog` collection. App-user continues to write `modificationOn` / `modificationBy` on patch, bulk, and delete while also calling `appUserAuditService` — mixed old/new patterns within one PR across sibling entity families.

Impact:
- Two sources of “who last changed this” for app-users vs one for crew/skillshow.
- List API shape differs across management domains; frontend types already dropped modification fields for crew/skillshow only.

Recommendation:
In this PR (preferred for consistency): remove app-user `modificationOn` / `modificationBy` writes and list projections mirroring the crew/skillshow diff, relying on audit logs only. If scoped out, mark follow-up ticket and add a short code comment on app-user service explaining the temporary dual path.

**PR comment (line 473, `app-user.service.ts` — new in this diff):** **High (Global consistency):** This PR adds `recordFormUpdate` / `recordBulkFormUpdates` / `recordCreated` on app-user but crew/skillshow also dropped `modificationOn` / `modificationBy` from models and repos. App-user still writes embedded modification metadata (e.g. lines 367–368, 502–505, 525–528) — migrate to audit-only (matching crew/skillshow) or document the intentional dual path.

**Status:** Accepted — intentional dual-tracking for app-user in this ticket; migration to audit-only deferred.
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Delete operations do not write audit entries | HIGH | Accepted | src/services/crew-user.service.ts | 249-255 | 249 |
| 2 | Audit persistence runs after DB commit without transaction | HIGH | Accepted | src/services/crew-user.service.ts | 184-196 | 190 |
| 3 | Duplicate reporting-manager snapshot logic (crew vs skillshow) | HIGH | Accepted | src/services/crew-user-audit.service.ts | 99-152 | 99 |
| 4 | App-user retains modificationOn/By; crew/skillshow migrated | HIGH | Accepted | src/services/app-user.service.ts | 367-368, 400-401, 454-456, 502-505, 525-528 (issue); 374, 473, 509 (PR-diff anchor) | 473 |

### Positive notes

- Clean layering: `*-audit.service` → `auditLogService` → `auditLogRepository` → `audit-log.utils` (`diffAuditSnapshots`, `pickAuditSnapshot`, `normalizeAuditValue`).
- Per-entity audit field allow-lists live in `src/constants/*.constants.ts`.
- Entity list routes validate params and use `admin` RBAC; audit routes registered before `/:id` catch-alls.
- Crew/skillshow bulk patches re-fetch list rows before diffing; reporting-manager IDs resolved to display names in audit snapshots.
- Partner sport/logo normalization avoids noisy or path-leaking audit diffs.
- Unit tests for `audit-log.utils`; domain service tests mock audit services to isolate behavior.

**Merge readiness:** **No open Critical/High blockers on the backend diff.** All four findings **Accepted** (deferred or intentional for SKSH-296 v1).
