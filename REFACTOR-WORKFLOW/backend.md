# PR review — REFACTOR-WORKFLOW (skillshow)

| Field | Value |
|-------|-------|
| PR | [#230](https://github.com/SkillshowFx/skillshow/pull/230) |
| Branch | `refactor-workflow` → `main` |
| Head | `ef9a854f` |
| Scope | Staging VM deploy script — correct SCP port flag |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |
| Re-reviewed | 2026-07-07 @ `ef9a854f` |

## GitHub comments

_No open inline findings._

## Findings

---
scp reused SSH_OPTS with `-p` instead of `-P` for remote port

Risk Level: HIGH
File Path: scripts/deploy-staging-vm.sh
Lines: 83-97

Description:
**DRY / correctness.** `SSH_OPTS` uses `-p "$SSH_PORT"` (valid for `ssh`). The prior `scp "${SSH_OPTS[@]}"` call passed the same `-p` flag to `scp`, where `-p` means preserve file timestamps — not port. `scp` requires capital `-P` for the remote port. With a non-default `SSH_PORT`, the upload step could fail (treating the port number as a path) or connect on the wrong port.

Impact:
- Staging zip upload via `scp` may fail or skip the configured port when `SSH_PORT` ≠ 22.
- Deploy workflow reports failure at archive upload despite valid SSH connectivity for other steps.

Recommendation:
✅ Fixed — introduce `SCP_OPTS` mirroring host-key and identity options but using `-P "$SSH_PORT"`. Keep `SSH_OPTS` with `-p` for `ssh` calls.

---
Production deploy no longer waits for ECS service stability

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 176-187

Description:
The workflow removes the `aws ecs wait services-stable` step and ends the production job after `ecs update-service --force-new-deployment`. The workflow reports success when the rollout is **triggered**, not when tasks pass health checks and the service reaches a steady state.

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
✅ Fixed on `41215491` — remote script runs `npm ci` in `RELEASE_DIR`, then `ln -sfn` to `current`.

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
✅ Fixed on `32f9a06a` — workflow passes `SSH_KNOWN_HOSTS`; deploy script pins known hosts before SSH/SCP.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | scp reused SSH_OPTS with `-p` instead of `-P` for remote port | HIGH | ✅ Fixed | scripts/deploy-staging-vm.sh | 83-97 |
| 2 | Production deploy no longer waits for ECS service stability | HIGH | Accepted | .github/workflows/deploy.yml | 176-187 |
| 3 | Staging `current` symlink flipped before `npm ci` succeeds | HIGH | ✅ Fixed | scripts/deploy-staging-vm.sh | 109-112 |
| 4 | SSH uses `StrictHostKeyChecking=accept-new` without pinned host key | MEDIUM | ✅ Fixed | scripts/deploy-staging-vm.sh | 76-81 |

**Merge readiness:** No open Critical/High blockers. PR #230 correctly separates `scp` port (`-P`) from `ssh` port (`-p`); safe to merge.
