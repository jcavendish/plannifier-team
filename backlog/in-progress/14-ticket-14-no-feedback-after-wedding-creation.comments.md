# Comments: Ticket 14

### Devon — 2026-03-04 10:48

Started Ticket 14. Read all relevant code: `ImportReviewPageClient.tsx`, `wedding-creator.ts`, `create-wedding/route.ts`, `extraction-schemas.ts`.

Root cause confirmed: success path in `ImportReviewPageClient.tsx` is gated on `result.data?.weddingId` being truthy. If the RPC returns but `weddingId` is empty/null, no toast and no navigation fires — `finally` just stops the spinner.

Plan is ready. Fix is 15 lines across 2 files:
1. Client: always fire toast on 200, navigate to weddings list if `weddingId` missing
2. Server: add defensive log to capture the raw RPC result when `wedding_event_id` is empty

Awaiting Joao's approval before implementing.
