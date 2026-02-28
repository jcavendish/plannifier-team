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
│   ├── 2026-02/            # Sprint archive
│   └── summary.md          # Year-to-date summary
├── docs/                   # Team documentation
├── team/                   # Team coordination files
└── README.md               # This file
```

## Workflow Overview

**Discord is minimal (brief notifications only)**
**PRs are the primary communication hub** (detailed comments, discussion, diffs)
**Ticket files stay lean** (spec + status + PR link)

### Creating a New Ticket

1. **Create** `.md` file in `backlog/open/` with format: `NN-ticket-slug.md`
2. **Use this template:**
   ```markdown
   # Ticket N: Clear Title

   **Status:** Open
   **Assignee:** —
   **Severity:** High|Medium|Low
   **Size:** XS|S|M|L|XL
   **Type:** Feature|Bug Fix|Refactor|Docs|Research
   **Depends On:** (other tickets if applicable)
   **PR:** (empty — filled when PR opens)

   ## Summary
   Brief overview of what needs to be done.

   ## Changes Required
   - What files need to change
   - Implementation approach
   - Key files affected

   ## Testing
   How to validate this is complete.
   ```
3. **Commit** and **push**
4. **Post Discord:** "New ticket: N. Details in ticket file."

---

## Development Workflow

### Phase 1: Devon Develops

1. **Pick** ticket from `backlog/open/`
2. **Move** to `backlog/in-progress/`, update:
   ```markdown
   **Status:** In Progress
   **Assignee:** Devon
   ```
3. **Implement** (see CLAUDE.md in main repo)
4. **When ready,** open **draft PR** against `staging`:
   ```bash
   gh pr create --base staging --draft --title "draft: [Ticket N] short desc" --body "See comments below"
   ```
5. **Post detailed PR comment** with:
   - What changed (summary)
   - How to test (reproduction steps)
   - Which areas might regress
   - Edge cases, known limitations

6. **Update ticket header:**
   ```markdown
   **PR:** https://github.com/.../pull/XXX
   ```
7. **Post Discord:** "Ticket N ready for QA. See PR #XXX"

### Phase 2: Qamar Tests

1. **Check** draft PR (linked in ticket)
2. **Read** Devon's detailed PR comment
3. **Run QA** (all flows, mobile, locales, edge cases)
4. **Post PR comment** with test results:
   ```
   ## QA Results

   **Status:** ✅ PASS / ❌ FAIL

   **Flows tested:**
   - Flow 1: ✅ PASS
   - Flow 2: ✅ PASS
   - Mobile: ✅ responsive at 390px

   **Environment:** Desktop, Mobile, en/pt-BR
   ```

5. **If PASS:**
   - Post Discord: "Ticket N QA passed. Ready for code review."
   - Leave PR in draft (Devon will convert after approval)

6. **If FAIL:**
   - Post detailed PR comment with each issue
   - Post Discord: "Ticket N QA failed. See PR #XXX"
   - Leave PR in draft
   - Devon fixes, pushes new commits → Qamar re-tests

### Phase 3: Code Review & Merge

1. **Joao** reviews PR (code + all comments)
2. **Approves** and merges to `staging`
3. **Post Discord:** "Ticket N merged"
4. **Devon** updates ticket:
   - Move from `in-progress/` → `done/`
   - Update **Status** → Done

### Archiving (End of Sprint)

1. Move `done/` tickets to `archive/2026-SXX/`
2. Create summary:
   ```
   ~/workspace/plannifier-team/archive/2026-SXX/summary.md
   ~/workspace/plannifier/docs/team/archive/2026-SXX/summary.md
   ```
3. Include: completed tickets, deferred items, decisions, metrics

---

## Current Backlog

### Open (8 tickets)

- **Ticket 1:** Prompt quality — extraction accuracy fixes (Batch F)
- **Ticket 2:** Installment pipeline end-to-end (Batch G, depends on #1)
- **Ticket 3:** Document attachment & vendor contracts (Batch G, depends on #1)
- **Ticket 4:** Vendor update-on-match instead of skip (Batch G, depends on #1)
- **Ticket 5:** Incremental import for existing weddings (P2, depends on #1-4)
- **Ticket 6:** Review UI redesign — swipable cards (UI Enhancement)
- **Ticket 7:** Housekeeping & observability (Polish)
- **Ticket 8:** Fix type-check OOM crash (Build Infra)

### In Progress

(See `/in-progress/`)

### Done

(See `/done/`)

---

## Communication

### Discord (Minimal)
- "Ticket N ready for QA. See PR #XXX"
- "Ticket N QA passed."
- "Ticket N merged."
- Emergencies / blockers only

### PRs (Primary Hub)
- Devon's detailed testing approach
- All QA results and issues
- Code review discussion
- Permanent record (archived with code)

### Ticket Files
- Specification (what + why)
- Status (Open/In Progress/Done)
- PR link (once opened)
- Dependencies + priority

---

## Related Repos

- **Main Codebase:** `~/workspace/plannifier` — Next.js 15, Supabase, TypeScript
- **Team Docs:** `~/workspace/plannifier/docs/team/` — STATE.md, IN_PROGRESS.md, DECISIONS.md

## Team

- **Polly** (`<@1476579040675496149>`) — Product Owner
- **Devon** (`<@1476576785822126151>`) — Senior Developer
- **Qamar** (`<@1476583149205979368>`) — QA Engineer
- **Joao** (`<@690532830975098882>`) — Tech Lead

**Last updated:** 2026-02-28
