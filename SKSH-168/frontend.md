# Frontend PR Review — skillshow-admin-ui (`SKSH-168`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-168`  
**Base:** `main...HEAD`  
**Scope:** Connect Social unlink confirmation modal (Critical, High, Medium)  
**Commits:** `7345e861` fix: 168 (+ merge from main)  
**Files changed:** 1 (`ConnectSocialModal.tsx`)

---

---
Destructive action appears above Cancel on mobile

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx  
Lines: 555-578

Description:
The confirm footer switched from `vertical={isMobile}` (natural column order) to `flex-col-reverse` with `flex-col-reverse items-stretch md:flex-row`. Below the `md` breakpoint, `flex-col-reverse` renders the second button (Unlink) above Cancel.

On `main`, mobile stacked Cancel first, then Disconnect — the safer pattern for destructive confirms.

Impact:
- On drawer/mobile layouts, the primary destructive control is the first tap target, increasing accidental unlink risk.
- Behavior diverges from the previous mobile layout without an obvious product reason.

Recommendation:
Drop `flex-col-reverse` so DOM order matches visual order on small screens (Cancel on top, Unlink below), or keep column order and only use `md:flex-row` for desktop:

```tsx
<Flex
  className="mt-2! w-full flex-col items-stretch md:flex-row md:items-center"
  gap="middle"
  justify="center"
>
```

Retain full-width buttons via `w-full!` / `md:w-auto!` as needed.

**PR comment (line 555):** On mobile, `flex-col-reverse` puts **Unlink** above **Cancel**; main had Cancel first. Consider a normal column stack so the destructive action stays below Cancel on small viewports.

---

---
Mixed “Disconnect” and “Unlink” copy in the same dialog

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx  
Lines: 373, 547-577

Description:
The confirmation modal title still uses “Disconnect” (`Disconnect ${confirmProvider.name} Account`), while the primary button label was changed to “Unlink”. The list-row control still exposes `aria-label="Disconnect"`.

Impact:
- Inconsistent terminology in one flow (title vs CTA vs screen reader label).
- Users and assistive tech may hear “Disconnect” while the visible action says “Unlink”.

Recommendation:
Align copy to one term (likely “Unlink” if that is the ticket spec):

```tsx
<Typography.Title ...>
  {confirmProvider ? `Unlink ${confirmProvider.name}?` : "Unlink account?"}
</Typography.Title>
// ...
<Button aria-label="Unlink" ... />
```

**PR comment (line 577):** This CTA is “Unlink” but the modal title (line 548) still says “Disconnect … Account” and the list-row control still uses `aria-label="Disconnect"` (line 373) — please align title, button, and `aria-label` to the same verb.

---

---
`disconnectWarning` has no fallback when provider lookup fails

Risk Level: MEDIUM  
File Path: src/pages/dashboard/components/ConnectSocialModal.tsx  
Lines: 164-168, 551-552

Description:
The per-platform `switch` was replaced with a message derived from `confirmProvider?.name`. If `confirmDisconnect` is set but `socialProviders.find` returns `undefined` (e.g. custom `providers` prop omitting an id, or list refresh while the modal is open), `disconnectWarning` is an empty string.

The previous `default` branch returned: “This will unlink the account. You can connect again anytime.”

Impact:
- Confirm modal can show a title with no explanatory body copy in edge cases.
- Weaker guardrail if provider config and connection state drift.

Recommendation:
Restore a generic fallback when `platformName` is missing:

```ts
const disconnectWarning = useMemo(() => {
  const platformName = confirmProvider?.name;
  if (!platformName) {
    return "You won't be able to share photos or videos to this platform until it's connected again.";
  }
  return `You won't be able to share photos or videos to ${platformName} until it's connected again.`;
}, [confirmProvider]);
```

**PR comment (line 166):** If `confirmProvider` is missing, the body is blank — consider keeping a generic fallback like the old `default` case.

---

## Positive notes

- Dynamic warning from `confirmProvider.name` matches product copy (“share photos or videos”) and stays correct for “X” vs legacy ids.
- `useMemo` dependency on `confirmProvider` is appropriate after dropping the `switch`.
- Full-width footer buttons via Tailwind (`w-full!` / `md:w-auto!`) and `[&_.ant-space-item]:w-full` improve layout without extra `block={isMobile}` props.
- `md:` breakpoint aligns with `useResponsiveModalLayout()` (`down("md")`).

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Destructive action above Cancel on mobile | MEDIUM | Open | src/pages/dashboard/components/ConnectSocialModal.tsx | 555-578 |
| 2 | Mixed “Disconnect” / “Unlink” copy | MEDIUM | Open | src/pages/dashboard/components/ConnectSocialModal.tsx | 373, 547-577 |
| 3 | No fallback when `confirmProvider` is missing | MEDIUM | Open | src/pages/dashboard/components/ConnectSocialModal.tsx | 164-168, 551-552 |

**Merge readiness:** Three open Medium items (mobile button order, copy alignment, warning fallback). No Critical/High blockers; safe to merge after product sign-off if Medium items are accepted as follow-ups.
