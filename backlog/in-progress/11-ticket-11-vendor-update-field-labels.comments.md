# Comments: Ticket 11 — Translate vendor update field names in import review badge

### Devon — 2026-03-03 12:02

Picked up ticket. Read the component and i18n files. Plan is ready.

**Approach:** Add `VENDOR_FIELD_LABELS` map inside `ImportReviewVendorsSection.tsx` (using `t()` for translations), transform `updateFields` before joining, and add 5 keys under `ImportReview.vendors.fields` in both locale files. Three files, ~15 lines total.

**Key finding:** `getVendorFieldsToUpdate()` only returns 4 DB keys (`contact_person`, `email`, `phone`, `service_category`) — `company_name` is in the map as a defensive fallback only (it's never gap-filled since the match is by name/email/phone).

Awaiting plan approval before implementing.
