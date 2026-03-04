# Ticket 18: Fix staging CI deploy — committed api/node_modules symlink breaks pnpm install

**Status:** Done
**Assignee:** Devon
**Severity:** Critical
**Size:** XS
**Type:** Bug Fix
**Depends On:** —
**Plan Status:** Approved
**PR:** https://github.com/jcavendish/plannifier/pull/347 (supersedes #345, #346)
**Latest Update:** #345→#346 fixed symlink issues. #347 fixes type-check OOM (Node heap limit). This should be the final CI blocker. Awaiting immediate merge.

## Summary

Every staging deploy has been failing since Feb 26 (the entire sprint). No code or migrations have reached the staging environment. Root cause: broken symlinks `api/node_modules` **and** `backend/node_modules` were accidentally committed to git.

Three CI blockers found in sequence:
1. `api/node_modules` committed symlink → fixed by PR #345
2. `backend/node_modules` committed symlink → fixed by PR #346
3. TypeScript type-check OOM (Node.js heap limit) → fixed by PR #347 (root `.npmrc` + explicit `NODE_OPTIONS` in CI)

Zero committed symlinks remain. #347 should be the final fix.

## Problem

`pnpm install --frozen-lockfile` in CI fails with:
```
ENOENT: no such file or directory, mkdir '/home/runner/.../api/node_modules'
```

`pnpm-workspace.yaml` declares `api` as a workspace package. pnpm tries to install its deps and symlink them into `api/node_modules`. But `api/node_modules` is a **committed symlink** pointing to `../../../../../api/node_modules` — a developer's local machine path that resolves to `/api/node_modules` in CI (which doesn't exist).

## Root Cause

Someone accidentally ran `git add api/node_modules` (or similar) on their machine, committing the symlink. The symlink is meaningless on any machine other than the original developer's.

## Fix

1. `git rm api/node_modules` — remove the committed symlink
2. Ensure `api/node_modules` is in `.gitignore`
3. pnpm handles all workspace installs through the root `.pnpm` virtual store — no separate `node_modules` per workspace package needed

## Acceptance Criteria

- [ ] `deploy-staging.yml` Quality Checks job passes
- [ ] Staging deploy completes successfully
- [ ] All pending migrations applied to staging Supabase
- [ ] Staging app is running latest code from the `staging` branch

## Latest Update

In progress — XS fix.
