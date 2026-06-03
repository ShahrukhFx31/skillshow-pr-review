# PR review workspace

Agent-driven PR reviews for Skillshow repos. **One ticket folder** groups reports across all services touched by the same branch/ticket.

## Layout

```
pr-review/
├── README.md
├── prompts/                          # Reusable review instructions (reference in chat)
│   ├── frontend-system-prompt.md       # skillshow-admin-ui
│   ├── backend-system-prompt.md        # skillshow
│   └── orchestrator-system-prompt.md   # skillshow-distribution-orchestrator
├── Completed/                        # Tickets with all findings Fixed or Accepted
│   └── {TICKET}/                     # Same layout as active tickets
└── {TICKET}/                         # Active reviews (uppercase id, e.g. SKSH-271)
    ├── frontend.md                   # skillshow-admin-ui review
    ├── backend.md                    # skillshow review (when run)
    └── orchestrator.md               # distribution orchestrator review (when run)
```

## Ticket folder naming

- Use the **Jira/ticket id in uppercase**: `SKSH-267`, `SKSH-271`
- Derive from branch name when needed (`feature/SKSH-271-foo` → `SKSH-271`)
- Do not mix casing (`sksh-271` → rename folder to `SKSH-271`)

## How to run a review

1. Open the PR diff for the target repo.
2. Reference the matching prompt, e.g. `@pr-review/prompts/frontend-system-prompt.md`.
3. Ask to review the PR and save output under `pr-review/{TICKET}/` using the repo filename (`frontend.md`, `backend.md`, or `orchestrator.md`).
4. Repeat for other repos on the same ticket so all reports sit in one `{TICKET}/` folder.
5. On re-review, update each finding’s **Status** in the summary table (`Open` → `Fixed` / `Accepted`).
6. When **every** summary row in **every** report for the ticket is `Fixed` or `Accepted` (no `Open`), move the folder to `pr-review/Completed/{TICKET}/` (see prompts → **Archive when complete**).

## Active tickets

Folders under `pr-review/{TICKET}/` (not yet in `Completed/`).

| Ticket   | Reports |
|----------|---------|
| SKSH-196 | `frontend.md`, `backend.md` |
| SKSH-265 | `frontend.md`, `backend.md` |
| SKSH-271 | `frontend.md`, `backend.md` |
| SKSH-274 | `frontend.md` |
| SKSH-294 | `frontend.md`, `backend.md` |
| SKSH-297 | `frontend.md`, `backend.md` |

## Completed tickets

Archived under `pr-review/Completed/` when all findings are **Fixed** or **Accepted** (no **Open** rows in any report for that ticket).

| Ticket   | Reports |
|----------|---------|
| SKSH-101 | `frontend.md`, `backend.md` |
| SKSH-116 | `frontend.md`, `backend.md` |
| SKSH-137 | `frontend.md`, `backend.md` |
| SKSH-168 | `frontend.md` |
| SKSH-178 | `frontend.md`, `backend.md` |
| SKSH-181 | `frontend.md`, `backend.md` |
| SKSH-189 | `frontend.md`, `backend.md` |
| SKSH-243 | `frontend.md` |
| SKSH-258 | `frontend.md` |
| SKSH-261 | `backend.md` |
| SKSH-267 | `frontend.md`, `backend.md`, `orchestrator.md` |
| SKSH-268 | `frontend.md` |
| SKSH-277 | `frontend.md`, `backend.md` |
| SKSH-278 | `frontend.md`, `backend.md` |
| SKSH-279 | `frontend.md` |
| SKSH-295 | `frontend.md`, `backend.md` |
| SKSH-302 | `frontend.md` |
| SKSH-305 | `frontend.md`, `backend.md` |
| SKSH-312 | `frontend.md`, `backend.md` |
| SKSH-273 | `frontend.md`, `backend.md` |
| SKSH-303 | `frontend.md`, `backend.md` |
| SKSH-311 | `frontend.md`, `backend.md` |
| SKSH-315 | `frontend.md` |
