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

### Devon — 2026-03-02

Implementation complete. PR #339 open as draft targeting staging. All 4 scope items delivered:

1. **Analytics** — 5 columns added to `document_imports` via migration; `process/route.ts` computes and persists after each run
2. **Token budget** — \$1.00 ceiling (env-configurable); aborts between file batches; warns at 80%; `ClaudeExtractionProvider` accumulates usage from every SDK call
3. **File retention** — `retention_expires_at` column (48h default) on `document_import_files`; new Vercel Cron at `/api/cron/cleanup-import-files` (3 AM UTC)
4. **Vendor heuristic** — `heuristicVendorMerge()` second pass in `extraction-merger.ts`; same-category + shared-contact merges zero-name-overlap vendors

Test status: 35/35 unit tests pass; build and lint clean. Reassigned to Qamar for QA.

### Qamar — 2026-03-02 10:20

**QA PASSED** — all 4 sub-tasks verified.

**Verification:**
- Build: clean ✅ | Lint: no new warnings ✅ | Type-check: clean ✅ | Unit tests: 967/967 ✅ (incl. 8 new heuristic tests)
- Code review: all changed files reviewed against diff

**Sub-task results:**
1. **Analytics** ✅ — Migration correct (IF NOT EXISTS, index), analytics computed and persisted in process/route.ts, response includes new stats fields
2. **Token budget** ✅ — Between-batch check, $0 guard for CLI provider, 80% warning fires once, accumulator resets per import
3. **File retention + cron** ✅ — 48h default, cron auth correct (401 on no/wrong secret), storage failure handled gracefully, vercel.json schedule correct
4. **Vendor heuristic** ✅ — Phone normalization handles BR formats, runs after Levenshtein pass, 8 unit tests cover all edge cases

**Issue noted:**
🟡 Medium — PR #339 contains commit `0df40cb` (Ticket 6 E2E test updates). **Must merge Ticket 6 (PR #337) before Ticket 7**, otherwise CI E2E tests will fail (card-stack selectors won't exist in old review screen). No app code issue — purely a merge order concern. Noted in PR comment.

**Verdict:** PASS. Ready for Joao's review. Merge after PR #337.
