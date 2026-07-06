# PR review — REFACTOR-WORKFLOW (skillshow)

| Field | Value |
|-------|-------|
| PR | [#216](https://github.com/SkillshowFx/skillshow/pull/216) |
| Branch | `refactor-workflow` → `main` |
| Scope | Simplify manual deploy workflow; staging VM zip-based releases; production Buildx + ECS trigger |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |

## GitHub comments

### `.github/workflows/deploy.yml` (line 173)

**HIGH** — Production deploy no longer waits for ECS service stability

### `scripts/deploy-staging-vm.sh` (line 97)

**HIGH** — Staging `current` symlink flipped before `npm ci` succeeds

### `scripts/deploy-staging-vm.sh` (line 71)

**MEDIUM** — SSH uses `StrictHostKeyChecking=accept-new` without pinned host key

## Findings

---
Production deploy no longer waits for ECS service stability

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 170-181

Description:
The PR removes the `aws ecs wait services-stable` step and ends the production job after `ecs update-service --force-new-deployment`. The workflow now reports success when the rollout is **triggered**, not when tasks pass health checks and the service reaches a steady state.

Impact:
- Failed or stuck ECS rollouts (bad image, task crash loop, capacity issues) will not fail the GitHub Actions run.
- Operators may assume production is healthy while the service is still draining old tasks or failing new ones.

Recommendation:
Restore a wait/verification step after `update-service`, e.g.:

```yaml
aws ecs wait services-stable \
  --cluster "${ECS_CLUSTER}" \
  --services "${ECS_SERVICE}" \
  --region "${AWS_REGION}"
```

If full wait is too slow, at minimum poll deployment rollout state and fail the job when the primary deployment ends in `FAILED`.

---
Staging `current` symlink flipped before `npm ci` succeeds

Risk Level: HIGH
File Path: scripts/deploy-staging-vm.sh
Lines: 95-103

Description:
After unzip, the script immediately runs `ln -sfn ... current` and only then executes `npm ci --omit=dev`. With `set -euo pipefail`, a failed `npm ci` aborts the script **after** `current` already points at the new release directory without installed dependencies.

Impact:
- A transient registry/network failure during `npm ci` can leave staging serving from a broken `current` release (no `node_modules`, pm2 not restarted or started against incomplete tree).
- Prior release remains on disk but is no longer active.

Recommendation:
Install dependencies before activating the release, then swap atomically:

```bash
cd "${APP_DIR}/releases/${RELEASE_NAME}"
npm ci --omit=dev --no-audit --no-fund
ln -sfn "${APP_DIR}/releases/${RELEASE_NAME}" "${APP_DIR}/current"
# pm2 reload/start using ${APP_DIR}/current
```

Optionally keep the previous `current` symlink until pm2 reload succeeds.

---
SSH uses `StrictHostKeyChecking=accept-new` without pinned host key

Risk Level: MEDIUM
File Path: scripts/deploy-staging-vm.sh
Lines: 71

Description:
**Security.** Deploy and rollback scripts connect with `-o StrictHostKeyChecking=accept-new`, which trusts the host key on first connect. On a compromised network or DNS hijack during the first connection, the runner could upload release artifacts or env content to an attacker-controlled host.

Impact:
- First-connection MITM could exfiltrate `STAGE_ENV` and deployment artifacts.
- Lower risk after host key is cached, but CI runners are ephemeral so `accept-new` applies on every run.

Recommendation:
Pin the staging VM host key via a repository secret (e.g. `SKILLSHOW_STAGE_VM_SSH_HOST_KEY`) and use `StrictHostKeyChecking=yes` with `UserKnownHostsFile` populated from that secret, or bake the known_hosts entry into the workflow before calling the script.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Production deploy no longer waits for ECS service stability | HIGH | Open | .github/workflows/deploy.yml | 170-181 |
| 2 | Staging `current` symlink flipped before `npm ci` succeeds | HIGH | Open | scripts/deploy-staging-vm.sh | 95-103 |
| 3 | SSH uses `StrictHostKeyChecking=accept-new` without pinned host key | MEDIUM | Open | scripts/deploy-staging-vm.sh | 71 |

**Merge readiness:** Blocked — restore production rollout verification and make staging release activation atomic after dependency install.
