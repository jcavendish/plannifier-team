### Devon — 2026-03-01 16:58

Picked up ticket. Created branch `feat/ticket-7-housekeeping-observability` from `origin/staging`.

Read: `extraction-merger.ts`, `entity-filter.ts`, `process/route.ts`, `create-wedding/route.ts`, `claude-cli-extraction-provider.ts`, `extraction-provider.ts`, existing migrations.

Spawned Opus sub-agent for implementation planning.

### Devon — 2026-03-01 17:00

Opus plan complete. Full plan saved at `07-ticket-7-plan.md` in this folder.

**Plan summary (4 independent sub-tasks, all in-scope for Size:S):**

1. **Analytics** — Add 5 columns to `document_imports` (entity_counts, avg_confidence, processing_time_ms, token_usage, estimated_cost_usd). Log after extraction completes in `process/route.ts`.

2. **Token budget ceiling** — Extend provider interface to return `tokenUsage + estimatedCostUsd`. Track cumulative cost per import in `process/route.ts`. Abort remaining files if $1 ceiling exceeded. Return `budgetWarning` in API response (UI deferred).

3. **File retention** — Add `retention_expires_at` (48h default) to `document_import_files`. New Vercel Cron route `/api/cron/cleanup-import-files` — deletes expired records from DB + files from Supabase Storage. Add cron to `vercel.json`. No new Supabase Edge Function needed (follows existing patterns).

4. **Vendor dedup heuristic** — Add `heuristicVendorMerge()` to `extraction-merger.ts` as a second pass after Levenshtein. Logic: same serviceCategory AND shared phone/email → merge. Catches legal-name vs brand-name matches Levenshtein misses.

**Deferred:** 80% warning UI, analytics dashboard, Edge Function cleanup variant.
**Complexity:** ~4-6h. No migrations blocked, no dependencies.

**Awaiting Joao's approval before implementation.**
