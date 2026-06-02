# Backend PR Review — skillshow (`SKSH-137`)

**Base:** `main...HEAD`  
**Branch:** `SKSH-137`  
**Reviewed:** 2026-05-26  
**Scope:** SkillShow user management API, username service extraction, partner delete / `connectMessage` / `hostCode`  
**Findings:** 4 open (**0 Critical**, **4 High**)

---

## Non-atomic create leaves orphan User on profile insert failure

---
Non-atomic create leaves orphan User on profile insert failure

Risk Level: HIGH
File Path: src/services/skillshow-user.service.ts
Lines: 65-87

Description:
`createSkillshowUser` calls `authRepository.createUser`, mutates and `save()`s the `User`, then calls `skillshowUserRepository.insertSkillshowUser`. There is no MongoDB session/transaction and no compensating delete if `insertSkillshowUser` fails (sequence error, validation, duplicate `user` ref, etc.).

Impact:
- Orphan `User` documents without a `SkillshowUser` profile
- Email may already be consumed; retries fail with "Email already exists"
- Admin UI shows inconsistent state until manual cleanup

Recommendation:
Wrap `createUser` + `insertSkillshowUser` in a `mongoose.startSession()` transaction, or delete the created `User` in a `catch` before rethrowing. Match the failure semantics used for resend (rollback temp password) where possible.

**PR comment (lines 65-87):** **High:** Create is two-phase without rollback—if `insertSkillshowUser` fails after `createUser`, you leave an orphan `User` and block the email on retry. Use a transaction or compensating delete.
---

## Username availability race (check-then-insert)

---
Username availability race (check-then-insert)

Risk Level: HIGH
File Path: src/services/skillshow-user.service.ts
Lines: 58-74

Description:
`usernameService.pickFirstAvailableUsername` reads taken names, then `authRepository.createUser` inserts. Concurrent creates with the same candidate set can both pass the check; `User` has a unique sparse index on `username`.

Impact:
- Intermittent 500s (`E11000 duplicate key`) instead of a controlled 400/retry
- Failed create after partial writes if error handling does not roll back the `User`

Recommendation:
Catch Mongo duplicate-key on `username` (and optionally `email`), retry with the next candidate (bounded attempts), or use an atomic upsert/find-and-assign pattern. Surface a clear 400 if all candidates are exhausted.

**PR comment (lines 58-74):** **High:** Username is chosen with a read-then-write race; concurrent invites can hit the unique `username` index. Add retry-on-duplicate or atomic assignment instead of failing with 500.
---

## Bulk patch performs sequential full updates (N+1)

---
Bulk patch performs sequential full updates (N+1)

Risk Level: HIGH
File Path: src/services/skillshow-user.service.ts
Lines: 176-193

Description:
`bulkPatchSkillshowUsers` loops `userIds` and awaits `patchSkillshowUser` per id. Each `patchSkillshowUser` loads skillshow user + linked user, saves, and re-runs the full list-row aggregation.

Impact:
- Bulk-updating N users costs O(N) round-trips and O(N) aggregation pipelines
- Timeouts or gateway limits when admins bulk-update tens of rows

Recommendation:
Add repository bulk helpers (`updateMany` on `SkillshowUser` + `User` for shared `department`/`status`), single audit stamp, and return refreshed rows via one aggregation with `$match: { skillshowUserId: { $in: [...] } }`.

**PR comment (lines 176-193):** **High:** `bulkPatchSkillshowUsers` calls full `patchSkillshowUser` in a loop—this is N+1 DB/aggregation work. Prefer batched updates + one refresh query for bulk department/status changes.
---

## Welcome email failure hidden on create

---
Welcome email failure hidden on create

Risk Level: HIGH
File Path: src/services/skillshow-user.service.ts
Lines: 94, 246-271

Description:
After a successful create, `void this.sendWelcomeEmail(user, tempPassword, false)` fires without awaiting. `sendWelcomeEmail` swallows errors when `throwOnFailure` is false (logs only). Resend uses `throwOnFailure: true` and returns 500 on SMTP failure—create does not.

Impact:
- Admin sees "created successfully" while the invite never arrives
- User has `isTempPassword: true` but no credentials email; support must use resend manually

Recommendation:
Await `sendWelcomeEmail` on create with `throwOnFailure: true`, or return `{ row, emailSent: false }` and surface a warning in the API response so the UI can prompt resend.

**PR comment (line 94):** **High:** Create fires welcome email in the background and ignores SMTP failures—admins get success while the user never receives credentials. Await send on create or return a partial-success flag like resend does.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Non-atomic create (orphan User) | HIGH | Open | `skillshow-user.service.ts` | 65-87 |
| 2 | Username check-then-insert race | HIGH | Open | `skillshow-user.service.ts` | 58-74 |
| 3 | Bulk patch N+1 | HIGH | Open | `skillshow-user.service.ts` | 176-193 |
| 4 | Silent welcome email on create | HIGH | Open | `skillshow-user.service.ts` | 94, 246-271 |

## Positive notes

- Clear layer split: controller → service → repository; Joi on routes; admin RBAC on all skillshow-user routes
- Soft-delete filters on list pipeline (`isDeleted: false` on profile and linked user)
- `resendWelcomeEmail` resets temp password and fails loudly when email cannot be sent
- Good test coverage for service/controller/validation paths
- Username suggestion logic centralized in `username.service` (also reused from `UserController`)
- Partner delete wired with admin auth and soft-delete in repository

## Out of scope / not raised

- `partner.repository` `softDeleteByPartnerId` uses `new Date()` vs `currentDate()` — consistency only
- Permission seed data for admin UI routes (backend permissions service, not in this diff)
