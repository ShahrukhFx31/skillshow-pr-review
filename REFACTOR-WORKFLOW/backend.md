# PR review — REFACTOR-WORKFLOW (skillshow)

| Field | Value |
|-------|-------|
| PR | [#231](https://github.com/SkillshowFx/skillshow/pull/231) |
| Branch | `refactor-workflow` → `main` |
| Head | `0118fe8e` |
| Scope | Staging deploy moved to appleboy SSH/SCP actions; deploy script retained but unused |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |
| Re-reviewed | 2026-07-07 @ `0118fe8e` |

## GitHub comments

### `.github/workflows/deploy.yml`

**Line 117** — **HIGH** — Staging SSH/SCP steps omit host key fingerprint verification

**Lines 134-137** — **HIGH** — Indented heredoc breaks `.env.dev` write on the VM

**Line 141** — **HIGH** — In-place dist copy before `npm ci`; no atomic release activation

**Line 114** — **HIGH** — Workflow bypasses `deploy-staging-vm.sh` while scripts still implement zip/archive model

## Findings

---
Staging appleboy SSH/SCP steps omit host key fingerprint verification

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 114-175

Description:
**Security regression.** PR #230 / `32f9a06a` pinned the staging VM host key via `SKILLSHOW_STAGE_VM_SSH_KNOWN_HOSTS` and `StrictHostKeyChecking=yes` in `deploy-staging-vm.sh`. This PR replaces that script with five `appleboy/ssh-action` and `appleboy/scp-action` steps that pass host, port, user, and key only — no `fingerprint` on any step. The workflow header still documents `SKILLSHOW_STAGE_VM_SSH_KNOWN_HOSTS`, but the secret is unused. Without fingerprint verification, ephemeral runners are vulnerable to SSH MITM on deploy.

Impact:
- Compromised or spoofed staging host could receive production-like secrets (`.env.dev`, SSH key use) and deploy artifacts.
- Prior host-pinning fix is effectively reverted for the active deploy path.

Recommendation:
Add `fingerprint: ${{ secrets.SKILLSHOW_STAGE_VM_SSH_FINGERPRINT }}` (SHA256 from `ssh-keygen -l -f /etc/ssh/ssh_host_*_key.pub | cut -d' ' -f2`) to **every** appleboy SSH and SCP step. Alternatively, keep calling `scripts/deploy-staging-vm.sh`, which already enforces known-host pinning. Do not leave `SKILLSHOW_STAGE_VM_SSH_KNOWN_HOSTS` documented but unused.

---
Indented heredoc in workflow breaks staging `.env.dev` write

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 134-137

Description:
The "Write staging env on VM" step uses an inline heredoc inside a YAML-indented `script: |` block. The closing `EOF` delimiter and every env line are emitted with leading spaces from YAML indentation. Bash requires an unindented closing delimiter for `<< 'EOF'`, and the heredoc body will include those spaces — producing a malformed `.env.dev` (keys prefixed with whitespace) or causing the heredoc to never terminate.

Impact:
- Staging deploy may fail at the env-write step, or write an unusable env file so the app cannot boot with correct config.
- Silent misconfiguration if only some lines parse.

Recommendation:
Avoid inline heredocs in indented workflow scripts. Pipe the secret safely, matching the prior pattern: `printf '%s\n' "$STAGE_ENV" | ssh … "cat > … && chmod 600 …"`, or use appleboy `envs` / a base64-encoded `script_stop` payload. At minimum use `<<-EOF` with tab-indented delimiters and no indented secret body.

---
In-place staging deploy copies `dist` before `npm ci` with no release isolation

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 141-175

Description:
**Global consistency / ops.** The workflow copies `dist/*` and manifests directly into `APP_DIR`, then runs `npm ci` in place and `pm2 reload`. Unlike the zip/release flow in `deploy-staging-vm.sh` (unzip to `releases/{name}`, `npm ci`, then flip `current`), a failed `npm ci` leaves new `dist` on disk with stale `node_modules`. `dist` is also overwritten while the prior process may still be serving traffic.

Impact:
- Transient `npm ci` failure can leave staging in a broken mixed state (new code, old deps).
- No retained archives for rollback (`rollback-staging-vm.sh.disabled` expects `archives/release-*.zip` that this path never creates).

Recommendation:
Either keep the release-directory + `current` symlink pattern (call `deploy-staging-vm.sh` or replicate its ordering), or add a staged directory: copy → `npm ci` in staging dir → atomic symlink swap / `pm2 reload` only after success. Restore archive retention if rollback remains a requirement.

---
Workflow bypasses `deploy-staging-vm.sh` while PR still updates zip-based scripts

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 114-175

Description:
**DRY / Global consistency.** The workflow no longer invokes `scripts/deploy-staging-vm.sh`, but the PR still modifies that script (unzip auto-install, `--ignore-scripts`) and `scripts/rollback-staging-vm.sh.disabled`. Two deploy models coexist: flat `APP_DIR/dist` via appleboy vs zip archives/releases in the shell scripts. Operators and docs reference different paths.

Impact:
- Dead code drift: script improvements never run in CI.
- Rollback script is disabled and incompatible with the new flat deploy.
- Future changes may update the wrong path.

Recommendation:
Pick one staging deploy implementation. If appleboy actions are intended, remove or clearly deprecate `deploy-staging-vm.sh` and align rollback docs. If zip/releases are intended, keep a single `bash scripts/deploy-staging-vm.sh` step and delete duplicate appleboy steps.

---
Production deploy no longer waits for ECS service stability

Risk Level: HIGH
File Path: .github/workflows/deploy.yml
Lines: 226-235

Description:
The workflow ends the production job after `ecs update-service --force-new-deployment` without `aws ecs wait services-stable`.

Impact:
- Failed or stuck ECS rollouts will not fail the GitHub Actions run.
- Operators verify rollout health outside the workflow.

Recommendation:
**Accepted (intentional).** Team prefers trigger-only production deploy; monitor ECS separately.

---
scp reused SSH_OPTS with `-p` instead of `-P` for remote port

Risk Level: HIGH
File Path: scripts/deploy-staging-vm.sh
Lines: 83-97

Description:
Fixed in PR #230 on the script path; no longer exercised when workflow uses appleboy SCP.

Impact:
- N/A for active deploy path in this PR.

Recommendation:
✅ Fixed on script path (`SCP_OPTS` with `-P`). Moot if workflow no longer calls the script.

---
Staging `current` symlink flipped before `npm ci` succeeds

Risk Level: HIGH
File Path: scripts/deploy-staging-vm.sh
Lines: 132-136

Description:
Fixed in the shell script (symlink after `npm ci`), but the active workflow path no longer uses release dirs or `current`.

Impact:
- Regression for the appleboy deploy path (in-place copy + reload).

Recommendation:
Partially fixed in script only; **Open** on workflow path — see finding #3.

---
SSH host key pinning via known_hosts (prior fix)

Risk Level: HIGH
File Path: scripts/deploy-staging-vm.sh
Lines: 76-88

Description:
**Security.** Script still pins host keys; workflow path does not.

Impact:
- See finding #1.

Recommendation:
**Open (regression)** on active workflow path.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Staging appleboy SSH/SCP steps omit host key fingerprint verification | HIGH | Open | .github/workflows/deploy.yml | 114-175 |
| 2 | Indented heredoc in workflow breaks staging `.env.dev` write | HIGH | Open | .github/workflows/deploy.yml | 134-137 |
| 3 | In-place staging deploy copies `dist` before `npm ci` with no release isolation | HIGH | Open | .github/workflows/deploy.yml | 141-175 |
| 4 | Workflow bypasses `deploy-staging-vm.sh` while PR still updates zip-based scripts | HIGH | Open | .github/workflows/deploy.yml | 114-175 |
| 5 | Production deploy no longer waits for ECS service stability | HIGH | Accepted | .github/workflows/deploy.yml | 226-235 |
| 6 | scp `-p` vs `-P` port flag | HIGH | ✅ Fixed | scripts/deploy-staging-vm.sh | 83-97 |
| 7 | Staging `current` symlink before `npm ci` | HIGH | Partially fixed | scripts/deploy-staging-vm.sh | 132-136 |
| 8 | SSH host key pinning (script path) | HIGH | Open | scripts/deploy-staging-vm.sh | 76-88 |

**Merge readiness:** **Not ready** — four open High findings on the new appleboy staging deploy path (host verification regression, broken env write, non-atomic deploy, dual deploy models).
