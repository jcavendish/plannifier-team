# Ticket 8: Fix type-check OOM crash during pnpm run test

**Status:** Open
**Assignee:** —
**Severity:** High
**Size:** M
**Type:** Bug Fix

## Summary

The TypeScript type-check phase during `pnpm run test` crashes with out-of-memory errors on systems with limited RAM, blocking CI/CD pipelines and local development. Requires optimization of type-checking configuration and/or tooling.

## Problem

- Running `pnpm run test` causes TypeScript type-check to consume unbounded memory
- Process crashes with OOM on VMs and CI systems with <4GB RAM
- Blocks validation gates and forces developers to work around the issue

## Root Causes to Investigate

1. **tsconfig.json configuration** — overly aggressive type-checking options
2. **Circular type dependencies** — causing exponential type inference
3. **Large generated types** — extraction schemas, Supabase-generated types
4. **Missing memory constraints** — no heap size limits for tsc process
5. **Incremental mode issues** — corrupted .tsbuildinfo files

## Changes Required

### 1. Analyze Type-Check Profile

- Run `tsc --diagnostics` to identify slowest/largest type-check phases
- Check for circular dependencies with `tsc --noEmit --listFiles`
- Profile memory usage during type-check

### 2. Optimize tsconfig.json

- Consider disabling aggressive type inference where not critical
- Enable `noUnusedLocals`, `noUnusedParameters` in dev mode to catch issues early
- Test with `skipLibCheck: true` (if acceptable for project)
- Verify `incremental: true` doesn't corrupt on partial failures

### 3. Add Memory Constraints

- Set Node.js max heap size in build scripts:
  ```bash
  NODE_OPTIONS="--max-old-space-size=2048" pnpm run test
  ```
- Document required system memory in README

### 4. CI/CD Optimization

- Split type-check into separate job with resource limits
- Consider using `tsc --noEmit --skipLibCheck` for faster iteration
- Cache .tsbuildinfo between runs

## Files Changed

- `tsconfig.json`
- `package.json` (scripts)
- `.npmrc` or shell config (memory env vars)
- CI workflow files (if applicable)

## Testing

- [ ] Runs on 2GB RAM VM without OOM
- [ ] Runs on CI with memory constraints
- [ ] Incremental builds still work
- [ ] No false negatives in type-checking

## Epic

Build Infrastructure

## Notes

- Blocks local development on resource-constrained systems
- Blocks CI pipelines when memory is limited
- Should be prioritized for developer experience
