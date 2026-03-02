# Ticket 9: Mobile layout — vendor name hidden behind contract badges

**Status:** In Progress
**Assignee:** Qamar
**PR:** https://github.com/jcavendish/plannifier/pull/336
**Severity:** Medium
**Size:** XS
**Type:** Bug Fix
**Depends On:** Ticket 3 (landed)

## Summary

On small viewports (≤390px), when a vendor contract card shows both the "Source document attached" badge AND a status badge simultaneously, the vendor name `<span>` collapses to width:0 and becomes invisible.

## Root Cause

In `ImportReviewVendorContractsSection.tsx`, the header row uses:
- Left side: `flex items-center gap-2 min-w-0` with `<span className="font-medium truncate">` for the vendor name
- Right side: `flex items-center gap-2 shrink-0` for the badges

The `shrink-0` on the right div prevents it from compressing, but when two badges are present on a narrow screen they consume all available width, leaving the left flex child (even with `min-w-0`) with zero effective space before truncation kicks in.

## Fix

Wrap badges in a `flex-wrap` container so they can break to a second line on small screens, or set a `max-w` cap on the badge container:

```tsx
{/* Option A — wrap badges */}
<div className="flex flex-wrap items-center gap-2 justify-end">
  {contract.sourceFileId && <Badge ...>...</Badge>}
  <Badge ...>status</Badge>
</div>

{/* Option B — cap badge container width */}
<div className="flex items-center gap-2 shrink-0 max-w-[60%]">
```

Option A is preferred — preserves all badges at the cost of a second line on mobile, which is acceptable.

## Files Changed

- `frontend/src/components/documents/ImportReviewVendorContractsSection.tsx`

## Found by

Qamar — QA Round 2 on Ticket 3 (non-blocking medium, deferred to follow-up)

## Epic

Import Feature — Polish
