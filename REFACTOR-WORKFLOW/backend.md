# PR review — REFACTOR-WORKFLOW (skillshow)

| Field | Value |
|-------|-------|
| PR | [#216](https://github.com/SkillshowFx/skillshow/pull/216) |
| Branch | `refactor-workflow` → `main` |
| Head | `32f9a06a` |
| Scope | Simplify manual deploy workflow; staging VM zip-based releases; production Buildx + ECS trigger |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |
| Re-reviewed | 2026-07-06 @ `32f9a06a` |

## GitHub comments

_No open inline findings._

## Findings

---
Production deploy no longer waits for ECS service stability

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 176-187

Description:
The PR removes the `aws ecs wait services-stable` step and ends the production job after `ecs update-service --force-new-deployment`. The workflow reports success when the rollout is **triggered**, not when tasks pass health checks and the service reaches a steady state.

Impact:
- Failed or stuck ECS rollouts will not fail the GitHub Actions run.
- Operators verify rollout health outside the workflow (ECS console, alarms, or manual follow-up).

Recommendation:
**Accepted (intentional).** Team prefers a fast trigger-only production job; monitor ECS deployment status separately rather than blocking the workflow on `services-stable`.

---
Staging `current` symlink flipped before `npm ci` succeeds

Risk Level: HIGH
File Path: scripts/deploy-staging-vm.sh
Lines: 109-112

Description:
Initial review: after unzip, the script symlinked `current` before `npm ci`, so a failed install could leave staging on a broken release.

Impact:
- (Resolved) Transient `npm ci` failures no longer repoint `current` before dependencies are installed.

Recommendation:
✅ Fixed on `41215491` — remote script runs `npm ci` in `RELEASE_DIR`, then `ln -sfn` to `current`. Rollback script updated with the same order.

---
SSH uses `StrictHostKeyChecking=accept-new` without pinned host key

Risk Level: MEDIUM
File Path: scripts/deploy-staging-vm.sh
Lines: 76-81

Description:
**Security.** Initial review: deploy script used `StrictHostKeyChecking=accept-new` on ephemeral runners.

Impact:
- (Resolved) Host key is now supplied via `SSH_KNOWN_HOSTS` / `SKILLSHOW_STAGE_VM_SSH_KNOWN_HOSTS` with `StrictHostKeyChecking=yes` and a dedicated `UserKnownHostsFile`.

Recommendation:
✅ Fixed on `32f9a06a` — workflow passes `SSH_KNOWN_HOSTS`; deploy and rollback scripts pin known hosts before SSH/SCP.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Production deploy no longer waits for ECS service stability | HIGH | Accepted | .github/workflows/deploy.yml | 176-187 |
| 2 | Staging `current` symlink flipped before `npm ci` succeeds | HIGH | ✅ Fixed | scripts/deploy-staging-vm.sh | 109-112 |
| 3 | SSH uses `StrictHostKeyChecking=accept-new` without pinned host key | MEDIUM | ✅ Fixed | scripts/deploy-staging-vm.sh | 76-81 |

**Merge readiness:** No open Critical/High/Medium blockers. ECS trigger-only production deploy accepted by team; staging atomic activation and SSH host pinning resolved.
