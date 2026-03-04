# Ticket 19: 529 overloaded_error not retried on classify/extract

**Status:** Done
**Assignee:** Devon
**Severity:** High
**Size:** XS
**Type:** Bug Fix
**Depends On:** —
**PR:** https://github.com/jcavendish/plannifier/pull/349
**Latest Update:** Merged (PR #349) and deployed to staging 2026-03-04. CI run 22678885360 — Quality Checks ✅ + Deploy to Staging ✅. No QA needed (1-line retryable error config change).

## Summary

Anthropic 529 `overloaded_error` responses were aborting classify/extract calls immediately (`canRetry: false`) instead of backing off and retrying. All 4 retry slots were wasted — the call failed on attempt 0 and bailed.

## Root Cause

`ANTHROPIC_RETRY_OPTIONS.retryableErrors` in `extraction-provider.ts` only listed `['429', 'rate_limit']`. The 529 error message contains `'529'` and `'overloaded'` — neither matched — so `isRetryable = false`. Anthropic's own response headers include `x-should-retry: true` on 529s.

## Fix

`frontend/src/lib/services/import/extraction-provider.ts`:
```ts
// Before:
retryableErrors: ['429', 'rate_limit'],

// After:
retryableErrors: ['429', 'rate_limit', '529', 'overloaded'],
```

1 file, 1 line. No migrations, no schema changes.
