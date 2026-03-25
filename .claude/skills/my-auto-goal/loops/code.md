# Code Domain Loop

Guidance for `domain_type: code` loops.

## Scope

Modify source code and run evaluations to improve a metric. No drift into research or unrelated refactoring.

## Quality Gate

Before keeping any iteration, ALL must pass:

| Check | Pass criteria |
|-------|--------------|
| Tests | `pytest` / `npm test` → exit 0 |
| Types | `mypy` / `tsc --noEmit` → no errors |
| Lint | `ruff` / `eslint` → no errors |
| Build | `uv build` / `npm run build` → exit 0 |

**Retry protocol:** Fix → alternative approach → simplify (max 3 attempts). After 3 hypothesis failures, escalate to user.

## Hypothesis Generation

Examine in order:
1. **Bottlenecks** — profile, find hot spots, optimize
2. **Code smells** — long functions, duplication, deep nesting
3. **Algorithm improvements** — O(n²)→O(n), caching, batching
4. **Coverage gaps** — uncovered branches, missing edge case tests

## Iteration Pattern

1. Identify ONE specific improvement
2. Implement the change
3. Run quality gate → run eval metric
4. Decision:
   - Pass + improved → `git add + commit` (KEEP)
   - Pass + no improve but simpler → `git add + commit` (KEEP)
   - Pass + no improve → `git restore . && git clean -fd` (DISCARD)
   - Fail → retry (max 3)
5. Log `✅ KEEP` / `❌ DISCARD` / `🔄 RETRY (N/3)`

## Git Workflow

```bash
# Before iteration: ensure clean
git status

# KEEP: commit with description
git add <files>
git commit -m "autogoal iter N: <description> (+X%)"

# DISCARD: revert everything
git restore . && git clean -fd
```
