
### Devon — 2026-03-04 12:05
PR #345 merged but CI immediately hit a second round:
```
ENOENT: no such file or directory, mkdir '.../backend/node_modules'
```
Same issue — `backend/node_modules` was also a committed symlink (→ `../../../../../backend/node_modules`). Opened PR #346 to remove it and add to .gitignore. Verified with `git ls-files -s | awk '$1 == "120000"'` — zero committed symlinks remain after this change. Awaiting immediate merge from Joao.
