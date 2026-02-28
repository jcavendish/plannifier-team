# Plannifier Team Management

This repository holds team management files and the backlog for the Plannifier wedding planning SaaS. It's kept separate from the main codebase to keep git commits focused on code changes only.

## Repository Structure

```
plannifier-team/
├── backlog/
│   ├── open/               # Not yet started
│   ├── in-progress/        # Active work (in flight)
│   ├── done/               # Completed and QA-passed
│   └── README.md           # Backlog workflow
├── archive/
│   ├── 2026-02/            # Sprint archive (monthly)
│   └── summary.md          # Year-to-date summary
├── docs/                   # Team documentation
├── team/                   # Team coordination files
└── README.md               # This file
```

## Workflow

### Creating a New Ticket

1. **Create** a new `.md` file in `backlog/open/` with format: `NN-ticket-slug.md`
2. **Create two files:**

   **Main file:** `NN-ticket-slug.md`
   ```markdown
   # Ticket N: Clear Title

   **Status:** Open
   **Assignee:** —
   **Severity:** High|Medium|Low
   **Size:** XS|S|M|L|XL
   **Type:** Feature|Bug Fix|Refactor|Docs|Research
   **Depends On:** (other tickets if applicable)

   ## Summary
   Brief overview of what needs to be done.

   ## Changes Required
   - What files need to change
   - Implementation approach
   - Key files affected

   ## Testing
   How to validate this is complete.

   ## Latest Updates

   See `NN-ticket-slug.comments.md` for full discussion.
   ```

   **Comments file:** `NN-ticket-slug.comments.md`
   ```markdown
   # Ticket N Comments

   (Agents add comments here as work progresses)
   ```
3. **Commit** and **push** to trigger awareness
4. **Mention** `<@1476579040675496149>` (Polly) in Discord with brief summary and link to ticket

### Working on a Ticket (Devon)

1. **Pick** a ticket from `backlog/open/`
2. **Move** it to `backlog/in-progress/`
3. **Update** the file header:
   ```markdown
   **Status:** In Progress
   **Assignee:** Devon
   ```
4. **Work** on the implementation (see CLAUDE.md in main repo)
5. **When ready for QA:** hand off to Qamar

### QA Testing (Qamar)

1. **Receive** ticket from Devon (still in `backlog/in-progress/`)
2. **Test** using your QA process
3. **If PASS:** Move to `backlog/done/`, update **Status** to "Done"
4. **If FAIL:** Update ticket with issues, move back to `backlog/open/`, notify Devon

### Archiving (End of Sprint)

At sprint end (usually end of month):

1. Move `backlog/done/` tickets to `archive/2026-0X/`
2. Create `archive/2026-0X/summary.md` with:
   - Sprint dates
   - Tickets completed (links)
   - Metrics (velocity, quality notes)
   - Blockers encountered

## Current Backlog

### Open (8 tickets)

- **Ticket 1:** Prompt quality — extraction accuracy fixes (Batch F, foundational)
- **Ticket 2:** Installment pipeline end-to-end (Batch G, depends on #1)
- **Ticket 3:** Document attachment & vendor contracts (Batch G, depends on #1)
- **Ticket 4:** Vendor update-on-match instead of skip (Batch G, depends on #1)
- **Ticket 5:** Incremental import for existing weddings (P2 deferred, depends on #1-4)
- **Ticket 6:** Review UI redesign — swipable per-type cards (UI Enhancement, depends on #2-3)
- **Ticket 7:** Housekeeping & observability (Polish, no dependencies)
- **Ticket 8:** Fix type-check OOM crash during pnpm run test (Build Infra)

### In Progress

(None)

### Done

(None)

## Related Repos

- **Main Codebase:** `~/workspace/plannifier` — Next.js 15, Supabase, TypeScript
- **Team Docs (in code repo):** `~/workspace/plannifier/docs/team/` — STATE.md, IN_PROGRESS.md, DECISIONS.md

## Team

- **Polly** (`<@1476579040675496149>`) — Product Owner, owns backlog priority
- **Devon** (`<@1476576785822126151>`) — Developer, picks tickets to implement
- **Qamar** (`<@1476583149205979368>`) — QA Engineer, validates implementations
- **Joao** (`<@690532830975098882>`) — Lead, approves plans, merges PRs

## Process

All team members follow the **3-phase async workflow** defined in their AGENTS.md files:

1. **Phase 1 (Sync):** Acknowledge immediately, then RETURN
2. **Phase 2 (Async):** Do heavy work in background
3. **Phase 3 (Async):** Post results and coordinate

This keeps Discord responsive and prevents timeout issues.

---

**Last updated:** 2026-02-27
