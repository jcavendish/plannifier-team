# Comments: Ticket 11 — Translate vendor update field names in import review badge

### Devon — 2026-03-03 12:02

Picked up ticket. Read the component and i18n files. Plan is ready.

**Approach:** Add `VENDOR_FIELD_LABELS` map inside `ImportReviewVendorsSection.tsx` (using `t()` for translations), transform `updateFields` before joining, and add 5 keys under `ImportReview.vendors.fields` in both locale files. Three files, ~15 lines total.

**Key finding:** `getVendorFieldsToUpdate()` only returns 4 DB keys (`contact_person`, `email`, `phone`, `service_category`) — `company_name` is in the map as a defensive fallback only (it's never gap-filled since the match is by name/email/phone).

Awaiting plan approval before implementing.

### Polly (approver) — 2026-03-03 13:00

Plan approved ✓

**Validation:**
- ✓ Solves the right problem: Badge shows raw DB column names instead of human-readable labels. Plan maps columns to translated strings via i18n keys.
- ✓ Scope reasonable: XS — 3 files, ~15 lines. No DB/API changes, pure label mapping.
- ✓ Edge cases covered: Plan notes `company_name` as defensive fallback (never actually returned by `getVendorFieldsToUpdate()`). No E2E test assertions needed since this is label-only.
- ✓ Business context: UX improvement for import review (couples/planners see "Contact person" instead of "contact_person"). Language-aware (en/pt-BR).

Ready for implementation on `feat/ticket-11-vendor-field-labels`.

### Devon — 2026-03-03 16:15

Implementation complete. Opened draft PR #342 against staging.

**What changed:**
- `ImportReviewVendorsSection.tsx`: added `VENDOR_FIELD_LABELS` map (5 DB keys → translated strings via `t()`), `updateFieldLabels` derived from it with `?? f` fallback
- `messages/en.json`: 5 keys added under `ImportReview.vendors.fields`
- `messages/pt-BR.json`: same 5 keys in Portuguese

**Lint:** 0 errors (86 pre-existing warnings, unchanged). TypeScript: clean.

**For QA:** Set up a vendor import where extracted vendor matches an existing catalog entry with a missing phone or contact person. Verify the "Will update" badge in the Vendors tab shows readable labels (`Contact person, Phone` in EN; `Pessoa de contato, Telefone` in pt-BR) instead of raw DB column names. No E2E assertions on label content exist, so this is a manual visual check.
