# App Domain Loop

Guidance for `domain_type: app` loops.

## Scope

Implement application features and verify with build/test. No unrelated research, no unapproved dependencies.

## Quality Gate

ALL must pass before keeping:

| Check | Pass criteria |
|-------|--------------|
| Build | `npm run build` / `uv build` → exit 0 |
| Tests | `npm test` / `pytest` → all pass |
| Types | `tsc --noEmit` / `mypy` → no errors |
| Lint | `eslint` / `ruff` → no errors |
| Runs | App starts without crash |

**Retry:** Fix → simpler approach → decompose smaller (max 3). After 3 hypothesis failures, ask user.

## Feature Implementation Phases

Build in order — don't skip phases:

```
Phase 1: Foundation — types, interfaces, data layer → verify build
Phase 2: Core Logic — business logic, unit tests → verify tests pass
Phase 3: Integration — UI components, wire to data, integration tests
Phase 4: Polish — error handling, loading states, edge cases
```

Each phase = 1+ iterations.

## Iteration Pattern

1. Identify ONE specific improvement
2. Implement the change
3. Run quality gate → run eval metric
4. Decision:
   - Pass + improved → commit (KEEP)
   - Pass + no improve but simpler → commit (KEEP)
   - Pass + worse → `git restore . && git clean -fd` (DISCARD)
   - Fail → retry (max 3)
5. Log `✅ KEEP` / `❌ DISCARD` / `🔄 RETRY (N/3)` / `⚠️ ESCALATE`
