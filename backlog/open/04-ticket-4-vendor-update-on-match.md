# Ticket 4: Vendor update-on-match instead of skip

**Status:** Open
**Assignee:** —
**Severity:** Medium
**Size:** S
**Type:** Feature
**Depends On:** Ticket 1

## Summary

When planner import detects existing vendor (by email/phone/name), update empty fields instead of skipping. Adopt admin import's 3-action model: create (new), update (fill gaps), match (no change).

## Problem

Currently, if a vendor already exists in planner's catalog, import silently skips it. Instead, merge strategy: fill empty fields without overwriting populated ones.

## Example

```
Existing vendor:   {name: "Cake Shop", email: "cake@shop.com", phone: null}
Imported vendor:   {name: "Cake Shop", email: null, phone: "11-99999-9999", address: "Rua X"}

Current behavior:  Skip entirely
New behavior:      Update to {name: "Cake Shop", email: "cake@shop.com", phone: "11-99999-9999", address: "Rua X"}
```

## Changes Required

### 1. conflict-resolver.ts

- Change return from `skip` to `update` when existing vendor has empty fields
- Compute merge payload (keep existing, fill from imported)

### 2. wedding-creator.ts

- Handle `update` action: build UPDATE payload
- Pass merge data to RPC

### 3. RPC (`create_wedding_from_import`)

- Add UPDATE logic: `UPDATE vendors SET phone = COALESCE(NULLIF(...), phone) WHERE id = ...`
- Use COALESCE to only update if imported value non-null

## Files Changed

- `conflict-resolver.ts`
- `wedding-creator.ts`
- Migration RPC

## Related Issues

- #19

## Reference

Reference admin import: `VendorDeduplicationService.ts`

## Epic

Import Feature - Batch G

## Notes

Parallel work: Can proceed with Tickets 2-3 after Ticket 1 is solid.
