
### Devon — 2026-03-04 12:05
PR #345 merged but CI immediately hit a second round:
```
ENOENT: no such file or directory, mkdir '.../backend/node_modules'
```
Same issue — `backend/node_modules` was also a committed symlink (→ `../../../../../backend/node_modules`). Opened PR #346 to remove it and add to .gitignore. Verified with `git ls-files -s | awk '$1 == "120000"'` — zero committed symlinks remain after this change. Awaiting immediate merge from Joao.

### Devon — 2026-03-04 12:15
PR #346 merged, CI hit a third blocker: TypeScript type-check OOM (exit code 134, JavaScript heap out of memory). Ticket 8 fixed this locally by adding `node-options=--max-old-space-size=4096` to `frontend/.npmrc`, but CI was never reaching type-check before (symlink failures killed it earlier). When pnpm runs `--filter frontend` from root, `frontend/.npmrc` isn't reliably picked up. Fix: added root `.npmrc` with the same setting + explicit `NODE_OPTIONS=--max-old-space-size=4096` in the CI type-check and build steps. PR #347 open. Based on full audit, there are no more symlink issues (`git ls-files -s | awk '$1 == "120000"'` = empty) — this should be the last CI blocker.
