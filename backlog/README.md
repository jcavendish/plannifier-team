# Backlog Organization

Tickets are organized by status in three folders:

## open/

**Status:** Not yet started

Tickets here are ready to be picked up by Devon. They may have dependencies (shown in "Depends On" field) that should be completed first.

**Naming convention:** `NN-ticket-slug.md` (e.g., `01-ticket-1-prompt-quality.md`)

**When to move out:** When Devon starts work → move to `in-progress/`

## in-progress/

**Status:** Currently being worked on

Tickets in this folder are actively being developed by Devon or tested by Qamar.

**When to move:**
- If work is paused: move back to `open/` with notes
- If work is complete and QA passes: move to `done/`

## done/

**Status:** Complete and QA-signed-off

Tickets here have passed testing and are ready to merge or have already been merged.

**At sprint end:** Archive these to `../archive/2026-0X/` with sprint summary.

## File Structure

Each ticket is a markdown file with this structure:

```markdown
# Ticket N: Title

**Status:** Open | In Progress | Done
**Assignee:** Devon | Qamar | Polly | Joao | —
**Severity:** High | Medium | Low
**Size:** XS | S | M | L | XL
**Type:** Feature | Bug Fix | Refactor | Docs | Research
**Depends On:** (Ticket 1, Ticket 2, etc. — leave blank if none)

## Summary
Brief overview.

## Changes Required
- What needs to change
- Implementation approach
- Key files

## Files Changed
List of files that will be modified.

## Related Issues
Related GitHub issue numbers.

## Testing
How to validate.
```

## Workflow Example

### Creating a ticket:
```bash
# Create in open/
echo "# Ticket X: ..." > backlog/open/XX-ticket-slug.md
git add backlog/open/XX-ticket-slug.md
git commit -m "feat: add Ticket X — brief description"
```

### Devon picks it up:
```bash
# Move to in-progress and update header
mv backlog/open/XX-ticket-slug.md backlog/in-progress/XX-ticket-slug.md
# Edit file: Status → In Progress, Assignee → Devon
git add -A
git commit -m "chore: move Ticket X to in-progress (Devon)"
```

### QA passes, move to done:
```bash
# Move to done
mv backlog/in-progress/XX-ticket-slug.md backlog/done/XX-ticket-slug.md
# Edit file: Status → Done
git add -A
git commit -m "chore: move Ticket X to done (QA passed)"
```

### End of sprint archive:
```bash
# Move all done/ to archive/2026-0X/
mkdir -p ../archive/2026-02
mv backlog/done/* ../archive/2026-02/
# Create summary
cat > ../archive/2026-02/summary.md << 'EOF'
# Sprint 2026-02 Summary
**Dates:** 2026-02-01 to 2026-02-28

## Completed
- Ticket X: ...
- Ticket Y: ...

## Metrics
- Velocity: N points
- Quality: N bugs found

## Blockers
- (none)
EOF
git add -A
git commit -m "chore: archive sprint 2026-02 tickets"
```

---

**Key principle:** File names and folder locations are the source of truth for ticket status. Always keep them in sync with the **Status** field in the file header.
