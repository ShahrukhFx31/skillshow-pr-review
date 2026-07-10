# PR review (SKSH-384) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#333](https://github.com/SkillshowFx/skillshow-admin-ui/pull/333) |
| Branch | `SKSH-384` → `main` |
| Head | `08211463946d141f5e22fe8ae2e359ca56dda7d8` |
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
- Sport sort helpers live in shared `src/utils/sport-options.utils.ts`.

## GitHub comments

_No open findings._

## Findings

---
Partner audit labels missing after contacts section move

Risk Level: HIGH
File Path: src/pages/partners/onboarding/components/partner-audit-log.tsx
Lines: 8-15

Description:
**Contract** / audit-log UI — `PARTNER_AUDIT_FIELD_LABELS` was built only from `FORM_SECTIONS` + `businessLogo`. Contact fields left `FORM_SECTIONS` for `PartnerContactsSection`, so `contacts` / legacy contact keys lacked display labels.

Impact:
- Audit UI showed raw keys instead of human labels.
- Violated `fieldLabels` ↔ backend audited field names contract.

Recommendation:
Extend `PARTNER_AUDIT_FIELD_LABELS` with explicit contact labels.

**Re-verify:** ✅ Fixed — labels added for `contacts`, `contactName`, `contactEmail`, `contactNumber`.
---

---
Shared sport sort helpers left under videos feature path

Risk Level: MEDIUM
File Path: src/utils/sport-options.utils.ts
Lines: 1-39

Description:
**DRY** / file structure — Sport A–Z helpers were imported across account, crew, partners, and share-account while living under `pages/videos/details/utils`.

Impact:
- Non-video features depended on a video-details util path.

Recommendation:
Move helpers to `src/utils/sport-options.utils.ts`.

**Re-verify:** ✅ Fixed — file renamed/moved to `src/utils/sport-options.utils.ts`; consumers import from `@/utils/sport-options.utils`.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Partner audit labels missing after contacts section move | HIGH | ✅ Fixed | `src/pages/partners/onboarding/components/partner-audit-log.tsx` | 8-15 |
| 2 | Shared sport sort helpers left under videos feature path | MEDIUM | ✅ Fixed | `src/utils/sport-options.utils.ts` | 1-39 |

**Merge readiness:** No open Critical/High/Medium blockers on frontend.
