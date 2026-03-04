# Comments: Ticket 14

### Devon — 2026-03-04 10:48

Started Ticket 14. Read all relevant code: `ImportReviewPageClient.tsx`, `wedding-creator.ts`, `create-wedding/route.ts`, `extraction-schemas.ts`.

Root cause confirmed: success path in `ImportReviewPageClient.tsx` is gated on `result.data?.weddingId` being truthy. If the RPC returns but `weddingId` is empty/null, no toast and no navigation fires — `finally` just stops the spinner.

Plan is ready. Fix is 15 lines across 2 files:
1. Client: always fire toast on 200, navigate to weddings list if `weddingId` missing
2. Server: add defensive log to capture the raw RPC result when `wedding_event_id` is empty

Awaiting Joao's approval before implementing.

### Devon — 2026-03-04 11:00

Implementation complete. PR #344 open as draft targeting staging.

Changes:
- `ImportReviewPageClient.tsx`: toast.success now always fires on 200. Navigate to wedding if weddingId present, fall back to /planner/weddings if empty.
- `wedding-creator.ts`: defensive console.error log when wedding_event_id is missing from RPC result.

Lint: 0 errors. Type-check: clean. Build: compiled and pages generated (trace step times out on sandbox — pre-existing, same as prior tickets).

Handing to Qamar for QA.
