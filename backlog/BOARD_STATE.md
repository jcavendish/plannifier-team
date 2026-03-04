# Board State
Updated: 2026-03-04T16:40:00.000Z

## Open (unassigned)

### Ticket 16: Vendor contracts not extracted — import review shows empty contracts tab
- **Assignee:** —
- **Severity:** High | **Size:** S | **Type:** Bug Fix
- **Depends On:** Ticket 15 ✅ (merged and deployed 2026-03-04)
- **File:** backlog/open/16-ticket-16-vendor-contracts-not-extracted-in-import.md
> Import review screen shows 0 vendor contracts even when source documents clearly contain contract data. Ticket 15 (rate limit fix) is now deployed — re-test on staging to see if contracts self-heal. If still empty, further debugging needed.

### Ticket 17: Source documents not linked to wedding after import creation
- **Assignee:** —
- **Severity:** Medium | **Size:** L | **Type:** Feature
- **Depends On:** Tickets 15, 16 (import pipeline stable)
- **File:** backlog/open/17-ticket-17-source-documents-not-linked-to-wedding.md
> Original uploaded PDF/document files are not attached to the wedding after creation. No UI to review/manage source documents inside a wedding. Needs scoping conversation before implementation.

## In Progress

*(no active tickets)*

## Done

### Ticket 15: Import rate limiting — 429 on concurrent file classification ✅
- **PR:** #348 merged 2026-03-04, staging deployed — CI run 22675945514 ✅ (Quality Checks + Deploy both passed)
- **File:** backlog/done/15-ticket-15-import-rate-limit-429-concurrent-classify.md

### Ticket 14: No user feedback after wedding creation ✅
- **PR:** #344 merged 2026-03-04, staging deployed (CI ✅)
- **File:** backlog/done/14-ticket-14-no-feedback-after-wedding-creation.md

### Ticket 19: 529 overloaded_error not retried on classify/extract ✅
- PR #349 merged 2026-03-04, staging deployed — CI run 22678885360 ✅
- 1-line fix: added '529' and 'overloaded' to ANTHROPIC_RETRY_OPTIONS.retryableErrors

### Ticket 18: Fix staging CI deploy pipeline ✅
- PRs #345 (api symlink), #346 (backend symlink), #347 (type-check OOM) — all merged 2026-03-04
- Staging deployed successfully. All Tickets 1–19 now live on staging.

### Ticket 1–13: Import feature sprint (all merged to staging ✅)
See individual done/ files for details.
