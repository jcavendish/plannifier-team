# Ticket 11: Translate vendor update field names in import review badge

**Status:** Done
**Assignee:** Review: Joao
**Severity:** Low
**Size:** XS
**Type:** Bug Fix
**Depends On:** —
**Plan Status:** Approved
**PR:** https://github.com/jcavendish/plannifier/pull/342
**Latest Update:** QA passed (Qamar, 2026-03-03) — code review + i18n verification. Build clean, all flows correct. Ready for Joao review.

## Summary

In the import review wizard (Vendors tab), when an extracted vendor matches an existing catalog entry and has fields to gap-fill, the badge reads "Vai atualizar: contact_person" or "Will update: phone" — raw DB column names instead of human-readable labels.

**Screenshot context:** "Vai atualizar: contact_person" and "Vai atualizar: phone" visible on vendor rows.

## Problem

`ImportReviewVendorsSection.tsx` computes:

```ts
const updateFields = Object.keys(getVendorFieldsToUpdate(vendor, existingCatalogVendor))
// → ['contact_person', 'phone']

t('vendors.catalogMatch.willUpdate', { fields: updateFields.join(', ') })
// → "Will update: contact_person, phone"  ← raw DB column names
```

`getVendorFieldsToUpdate()` (in `conflict-resolver.ts`) returns keys matching the DB column names: `company_name`, `contact_person`, `email`, `phone`, `service_category`.

## Root Cause

No mapping from DB column name → human-readable label exists. The `updateFields` array is joined and injected directly into the i18n string.

## Proposed Fix

Add a column-label map in `ImportReviewVendorsSection.tsx` (or a shared util) and transform before display:

```ts
const VENDOR_FIELD_LABELS: Record<string, string> = {
  company_name:     t('vendors.fields.companyName'),
  contact_person:   t('vendors.fields.contactPerson'),
  email:            t('vendors.fields.email'),
  phone:            t('vendors.fields.phone'),
  service_category: t('vendors.fields.serviceCategory'),
}

const updateFieldLabels = updateFields
  .map(f => VENDOR_FIELD_LABELS[f] ?? f)  // fallback to raw key if unknown
  .join(', ')
```

Add 5 new i18n keys to `en.json` and `pt-BR.json` under `ImportReview.vendors.fields`:

| Key | en | pt-BR |
|-----|----|-------|
| `companyName` | Company name | Nome da empresa |
| `contactPerson` | Contact person | Pessoa de contato |
| `email` | Email | E-mail |
| `phone` | Phone | Telefone |
| `serviceCategory` | Service category | Categoria de serviço |

## Files to Change

- `frontend/src/components/documents/ImportReviewVendorsSection.tsx` — add field label map + transform
- `frontend/messages/en.json` — 5 new keys under `ImportReview.vendors.fields`
- `frontend/messages/pt-BR.json` — 5 new keys (translated)

## Complexity

XS — no DB changes, no API changes, no new components. Pure label mapping + i18n strings.
