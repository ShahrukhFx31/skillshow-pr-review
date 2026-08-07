# Orchestrator PR Review — skillshow-distribution-orchestrator (`INSTALOGIN`)

**Repo:** SkillshowFx/skillshow-distribution-orchestrator  
**PR:** https://github.com/SkillshowFx/skillshow-distribution-orchestrator/pull/30  
**Branch:** `feat/insta-login` → main  
**Head:** `f023441faad028f1dd93932bfb8c95c49809ef0e`  
**Scope:** Switch Instagram Graph host to `graph.instagram.com`, bump API version, prefer `metadata.accountId`, in-memory test harness refactor  
**Prompt:** `pr-review/prompts/orchestrator-system-prompt.md`  
**Paired:** `pr-review/INSTALOGIN/backend.md` (#260), `pr-review/INSTALOGIN/frontend.md` (#372)  
**Updated:** 2026-08-07 — re-verify on latest head (new `.env.dev` blocker)

## GitHub comments

### `.env.dev`

- **L47** — Live AWS credentials committed in tracked env file

## Findings

---
Live credentials in tracked `.env.dev`

Risk Level: CRITICAL
File Path: .env.dev
Lines: 12-13, 28-30, 47-48, 67, 71, 75, 77

Description:
**Security.** The PR updates tracked `.env.dev` with live secrets: MongoDB URI (including a commented alternate URI with password), Gmail app password (`EMAIL_PASSWORD`), AWS access key/secret, Instagram/Facebook `CLIENT_SECRET`, YouTube `CLIENT_SECRET`, and `SERVER_TOKEN`. These must not live in git history — rotate any exposed credentials and use placeholders or local-only untracked env files.

Impact:
- Credential leak to anyone with repo access; violates pre-release secrets policy.
- Commented Mongo URI still exposes another cluster password in the diff.

Recommendation:
Revert `.env.dev` to placeholder values (or remove from the PR entirely). Keep real dev secrets in untracked `.env.local` / team secret store. Rotate AWS keys, OAuth client secrets, email app password, and DB passwords that appeared in this diff.
---

**Positive notes:** Instagram vendor code changes (`graph.instagram.com`, API v22, `accountId` resolution order) align with backend #260. In-memory test harness refactor is scoped appropriately.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Live credentials in tracked `.env.dev` | CRITICAL | Open | .env.dev | 12-13, 28-30, 47-48, 67, 71, 75, 77 |

**Merge readiness:** Request changes — revert/placeholder `.env.dev` and rotate exposed secrets before merge. Backend #260 and frontend #372 are otherwise merge-ready.
