# Board State
Updated: 2026-03-04T13:00:00.000Z

## Open (unassigned)

### Ticket 16: Vendor contracts not extracted — import review shows empty contracts tab
- **Assignee:** —
- **Severity:** High | **Size:** S | **Type:** Bug Fix
- **Depends On:** Ticket 15 (rate limit fix — now in PR #348)
- **File:** backlog/open/16-ticket-16-vendor-contracts-not-extracted-in-import.md
> Import review screen shows 0 vendor contracts even when source documents clearly contain contract data. Suspected root cause: 429 rate limit (Ticket 15) aborts extraction before contracts are parsed. Re-verify after Ticket 15 merges.

### Ticket 17: Source documents not linked to wedding after import creation
- **Assignee:** —
- **Severity:** Medium | **Size:** L | **Type:** Feature
- **Depends On:** Tickets 14, 15, 16 (import pipeline stable)
- **File:** backlog/open/17-ticket-17-source-documents-not-linked-to-wedding.md
> Original uploaded PDF/document files are not attached to the wedding after creation. No UI to review/manage source documents inside a wedding. Needs scoping conversation before implementation.

## In Progress

### Ticket 14: No user feedback after wedding creation
- **Assignee:** Devon (QA by Qamar ✅ — awaiting Joao merge)
- **Plan Status:** Approved
- **Latest Update:** QA passed (Qamar, 2026-03-04) — 4/4 E2E pass. PR #344 converted to ready-for-review. Awaiting Joao merge + staging deploy.
- **Severity:** High | **Size:** XS | **Type:** Bug Fix
- **PR:** https://github.com/jcavendish/plannifier/pull/344 (ready for review)
- **File:** backlog/in-progress/14-ticket-14-no-feedback-after-wedding-creation.md

### Ticket 15: Import rate limiting — 429 on concurrent file classification
- **Assignee:** Devon
- **Plan Status:** Approved
- **Latest Update:** Implementation complete, PR #348 open as draft. Awaiting Qamar QA.
- **Severity:** High | **Size:** S | **Type:** Bug Fix
- **PR:** https://github.com/jcavendish/plannifier/pull/348 (draft)
- **File:** backlog/in-progress/15-ticket-15-import-rate-limit-429-concurrent-classify.md

## Done

### Ticket 18: Fix staging CI deploy pipeline ✅
- PRs #345 (api symlink), #346 (backend symlink), #347 (type-check OOM) — all merged 2026-03-04
- Staging deployed successfully. All Tickets 1–14 now live on staging.

### Ticket 1–13: Import feature sprint (all merged to staging ✅)
See individual done/ files for details.
