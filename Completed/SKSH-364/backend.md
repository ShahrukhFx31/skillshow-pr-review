# PR review (SKSH-364) — skillshow

PR: `https://github.com/SkillshowFx/skillshow/pull/222`  
Base: `main`  
Head: `SKSH-364` @ `147e8a55`

Prompt: `pr-review/prompts/backend-system-prompt.md`

## GitHub comments

No inline comments — no Open Critical/High findings.

## Findings

No Critical or High findings.

### Positive notes

- **DRY:** Centralizes admin provisioning welcome copy/dispatch in `provisioning-welcome.utils.ts` (`sendProvisioningWelcomeInApp`, `notifyAdminProvisionedWelcomeInApp`) and reuses shared title constants.
- **Clear channel split:** Admin/import flows add in-app welcome alongside existing email; athlete add-athlete flow uses dedicated `notifyCreatedAthleteWelcome` (in-app only) without duplicating credential emails.
- **Idempotency:** Uses `buildBulkUserIdempotencyKey` / `buildConnectionIdempotencyKey` with stable batch keys (`app-user:{id}`, `import:{id}`).
- **Optional `channels` on relation notify:** Extends `RelationNotifyInput` without breaking default email+in-app behavior.
- **Tests:** Covers admin in-app welcome, athlete welcome copy variants, athlete create side-effect wiring, and notification copy utils.

### Optional follow-up (not reported as findings)

- `crew-user` / `skillshow-user` create paths still email-only; wire `notifyAdminProvisionedWelcomeInApp` if SKSH-364 scope includes all admin provisioning surfaces (out of this PR diff).

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No Critical/High findings | — | — | — | — |

**Merge readiness:** No open Critical/High blockers.
