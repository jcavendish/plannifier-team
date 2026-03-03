# Plan: Ticket 11 — Translate vendor update field names in import review badge

## Summary

XS fix. `getVendorFieldsToUpdate()` returns raw DB column keys (`contact_person`, `email`, `phone`, `service_category`). These are joined and rendered directly into the "Will update: {fields}" badge. We need a label map to translate them to human-readable strings before display.

## Files to Change

### 1. `frontend/src/components/documents/ImportReviewVendorsSection.tsx`

Add a `VENDOR_FIELD_LABELS` constant (inside the component, after the `t` call so it can use translations):

```ts
const VENDOR_FIELD_LABELS: Record<string, string> = {
  company_name:     t('vendors.fields.companyName'),
  contact_person:   t('vendors.fields.contactPerson'),
  email:            t('vendors.fields.email'),
  phone:            t('vendors.fields.phone'),
  service_category: t('vendors.fields.serviceCategory'),
}
```

Transform `updateFields` before the `t('vendors.catalogMatch.willUpdate', ...)` call:

```ts
const updateFieldLabels = updateFields
  .map(f => VENDOR_FIELD_LABELS[f] ?? f)  // fallback to raw key if unknown
  .join(', ')

// …
t('vendors.catalogMatch.willUpdate', { fields: updateFieldLabels })
```

### 2. `frontend/messages/en.json`

Add under `ImportReview.vendors`:

```json
"fields": {
  "companyName":     "Company name",
  "contactPerson":   "Contact person",
  "email":           "Email",
  "phone":           "Phone",
  "serviceCategory": "Service category"
}
```

### 3. `frontend/messages/pt-BR.json`

Add under `ImportReview.vendors`:

```json
"fields": {
  "companyName":     "Nome da empresa",
  "contactPerson":   "Pessoa de contato",
  "email":           "E-mail",
  "phone":           "Telefone",
  "serviceCategory": "Categoria de serviço"
}
```

## Notes

- `getVendorFieldsToUpdate` only ever returns `contact_person`, `email`, `phone`, `service_category` — `company_name` is included in the map as a defensive fallback only.
- No DB changes, no migrations, no API changes.
- No new E2E tests needed — this is a pure label change. Existing vendor-update E2E tests already check badge visibility; label content is not asserted.
- Build + lint + existing tests should pass without changes.

## Complexity

XS — 3 files, ~15 lines total.

## Approach

Approved plan will be implemented in a single commit on `feat/ticket-11-vendor-field-labels`.
