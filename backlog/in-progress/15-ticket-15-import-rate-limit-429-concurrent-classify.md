# Ticket 15: Import rate limiting — 429 on concurrent file classification

**Status:** In Progress
**Assignee:** Devon
**PR:** https://github.com/jcavendish/plannifier/pull/348
**Latest Update:** Implementation complete, PR #348 open as draft. Awaiting Qamar QA.
**Severity:** High
**Size:** S
**Type:** Bug Fix
**Depends On:** —

## Summary

When importing multiple files, the classify step fails with HTTP 429 (rate limit exceeded). Both concurrent classify calls hit the Anthropic API simultaneously, exhaust the 50K input tokens/minute org limit together, and then retry at the same time — causing repeated failures and degraded/failed imports.

## Problem

`processAllFiles` in `src/app/api/import/process/route.ts` processes files in batches of `MAX_CONCURRENCY=2` using `Promise.all`. Each file triggers two API calls: `classify` + `extract`. When two large PDFs are classified at the same time:

1. Both calls consume ~25K+ tokens each → total > 50K/min limit
2. Both fail with 429 simultaneously
3. `ANTHROPIC_RETRY_OPTIONS` uses fixed `backoffMs: 15_000` (no jitter)
4. Both calls retry at exactly the same time → both fail again

The org limit at the time of failure was 50K input tokens/min with 0 remaining. The `retry-after` header from Anthropic says 13s, but our backoff is hardcoded 15s and doesn't read that header.

## Root Cause

- `MAX_CONCURRENCY=2` causes concurrent large-PDF classification → token limit exceeded
- Fixed backoff with no jitter causes synchronized retries → repeated failures
- `ANTHROPIC_RETRY_OPTIONS.backoffMs` ignores the `retry-after` header from the 429 response

## Proposed Fix

1. **Reduce `MAX_CONCURRENCY` to `1`** for the classify+extract pipeline (serialize per file). This is the safest fix — eliminates the concurrency problem entirely for file-level processing. Two concurrent extractions per minute is already within the limit for most document sizes.
2. **Read `retry-after` from 429 response headers** in `handleOperationError` (or in `ClaudeExtractionProvider.classify`) and use that value as the actual backoff instead of the hardcoded 15s.
3. **Add exponential backoff with jitter** as a secondary defense so that if concurrency is ever increased again, retries are staggered.

## Acceptance Criteria

- [ ] Importing 2+ PDF files completes without 429 errors under normal conditions
- [ ] If a 429 does occur, backoff respects the `retry-after` header from Anthropic
- [ ] Retry logic uses jitter to prevent synchronized retries
- [ ] `MAX_CONCURRENCY` change is documented with a comment explaining the rate limit reasoning

## Files

- `frontend/src/app/api/import/process/route.ts` — `MAX_CONCURRENCY`, `processAllFiles`
- `frontend/src/lib/services/import/extraction-provider.ts` — `ANTHROPIC_RETRY_OPTIONS`
- `frontend/src/lib/error-handling.ts` — `handleOperationError` retry logic

## Latest Update

Open, not yet assigned.
