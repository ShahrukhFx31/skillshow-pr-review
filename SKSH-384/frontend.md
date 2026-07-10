# PR review (SKSH-384) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#333](https://github.com/SkillshowFx/skillshow-admin-ui/pull/333) |
| Branch | `SKSH-384` → `main` |
| Head | `15b7f2b8e7336a455d5538d97b92a88e0d040c52` |
| Scope | Default table page size → 20; A–Z sport/event-type ordering; partner multi-contact UI; host-code dashes; roles/permissions default name sort |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, audit-log UI stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Positive notes

- `DEFAULT_TABLE_PAGE_SIZE` / `DEFAULT_LIST_PAGINATION` / `DEFAULT_LIST_QUERY` / `DEFAULT_TABLE_PAGINATION` now share one source (20) and match backend `LIST_QUERY_PAGINATION`.
- Connections panel options `[10, 20, 50]` align with backend `ATHLETE_CONNECTIONS_PANEL`.
- Edit-request list correctly wires `showSizeChanger` to existing `(page, pageSize)` handler.
- Partner contacts section uses Form.List + shared empty/map helpers; destructive confirm patterns untouched.

## GitHub comments

### Summary-only (file not in PR diff)
- **HIGH** — Partner audit labels missing after contacts section move (`partner-audit-log.tsx` 8-11)

### `src/pages/videos/details/utils/sportOptions.ts`
- **MEDIUM** — Shared sport sort helpers left under videos feature path (line 12)

## Findings

```markdown
---
Partner audit labels missing after contacts section move

Risk Level: HIGH
File Path: src/pages/partners/onboarding/components/partner-audit-log.tsx
Lines: 8-11

Description:
**Contract** / audit-log UI — `PARTNER_AUDIT_FIELD_LABELS` is built only from `FORM_SECTIONS` + `businessLogo`. Contact fields were removed from `FORM_SECTIONS` and replaced by `PartnerContactsSection`, so `contacts`, `contactName`, `contactEmail`, and `contactNumber` no longer have display labels. Backend now audits `contacts` (and still audits legacy contact fields when synced).

Impact:
- Audit UI shows raw keys (`contacts`, `contactName`, …) instead of human labels.
- Violates the `fieldLabels` ↔ backend audited field names contract for the new multi-contact model.

Recommendation:
Extend `PARTNER_AUDIT_FIELD_LABELS` with explicit entries, e.g. `contacts: "Contacts"`, `contactName: "Contact Name"`, `contactEmail: "Contact Email"`, `contactNumber: "Contact Number"`, matching the form copy.
---
```

```markdown
---
Shared sport sort helpers left under videos feature path

Risk Level: MEDIUM
File Path: src/pages/videos/details/utils/sportOptions.ts
Lines: 5-38

Description:
**DRY** / file structure — `compareSportOptionLabels`, `sortSportSelectOptionsByLabel`, and `sortSportLabels` are now imported from account general, app-user onboarding, crew constants, partners, share-account, and `accountService`. Keeping cross-feature shared sort logic under `pages/videos/details/utils` creates a misleading dependency and invites further copy-paste.

Impact:
- Non-video features depend on a video-details util path.
- Harder to discover the single source of truth for sport ordering.

Recommendation:
Move the sort helpers (and ideally `filterSportSelectOptions` if still shared) to something like `src/utils/sport-options.ts` or `src/constants/sport-options.ts`, and re-export from the videos path only if needed for a short migration.
---
```

PR comments (inline, 2–4 sentences):

1. `PARTNER_AUDIT_FIELD_LABELS` no longer includes contact fields after `CONTACT_FIELDS` left `FORM_SECTIONS`. Add explicit labels for `contacts` / legacy contact keys so audit lines match the multi-contact UI and backend `PARTNER_AUDIT_FIELDS`.

2. Sport A–Z helpers are now used across account, crew, partners, and share-account but still live under `pages/videos/details/utils`. Promote them to a shared `src/utils` (or constants) module so non-video features do not depend on the videos tree.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Partner audit labels missing after contacts section move | HIGH | Open | `src/pages/partners/onboarding/components/partner-audit-log.tsx` | 8-11 |
| 2 | Shared sport sort helpers left under videos feature path | MEDIUM | Open | `src/pages/videos/details/utils/sportOptions.ts` | 5-38 |

**Merge readiness:** Request changes — 1 open High (partner audit labels). Medium sport-util placement should be cleaned up in the same PR if practical.
