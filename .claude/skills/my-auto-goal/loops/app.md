# App Domain Loop

Domain-specific guidance for `domain_type: app` loops.

## Scope

Implement application features and verify with build/test.

**Allowed:**
- Add new features
- Fix bugs
- Modify UI/UX
- Update configurations
- Add/modify tests
- Refactor existing code

**Not allowed:**
- Unrelated research loops
- Installing unapproved dependencies
- Modifying deployment configs without approval
- Changes outside the app scope

## Quality Gate

Before keeping any iteration, ALL must pass:

| Check | Command | Pass Criteria |
|-------|---------|---------------|
| Build | `npm run build` / `uv build` | Exit code 0 |
| Tests | `npm test` / `pytest` | All pass |
| Types | `tsc --noEmit` / `mypy` | No errors |
| Lint | `eslint` / `ruff` | No errors |
| App runs | Manual or `npm run dev` | No crash |

**Gate failure handling:**
- FAIL → Retry with fix (max 3 attempts)
- BLOCKED → Escalate to user
- After 3 retries on same issue, move to next hypothesis

## Typical Metrics

| Metric | Direction | Example eval_command |
|--------|-----------|---------------------|
| Build success | higher-is-better | `npm run build && echo "exit_code=0"` |
| Test pass | higher-is-better | `npm test -- --passWithNoTests` |
| Bundle size | lower-is-better | `npm run build && du -sk dist/` |
| Lighthouse score | higher-is-better | `npx lighthouse URL --output=json` |
| Type coverage | higher-is-better | `npx type-coverage` |
| E2E tests | higher-is-better | `npx playwright test` |

## Feature Implementation Pattern

For feature work, follow this sequence:

```
Phase 1: Foundation
- Add types/interfaces
- Add data layer (API, state)
- Verify build passes

Phase 2: Core Logic
- Implement business logic
- Add unit tests
- Verify tests pass

Phase 3: UI/Integration
- Add UI components
- Wire up to data layer
- Add integration tests

Phase 4: Polish
- Error handling
- Loading states
- Edge cases
```

Each phase = 1+ iterations. Don't skip phases.

## Iteration Pattern

```
1. Understand current app state
2. Run baseline eval (build + test)
3. Identify ONE specific improvement
4. Implement the change
5. Run quality gate
6. Run eval metric
7. Decision:
   - PASS + improved → git add + commit (KEEP)
   - PASS + no improve but simpler → git add + commit (KEEP)
   - PASS + no improve and not simpler → git restore . && git clean -fd (DISCARD)
   - FAIL → retry (max 3) or escalate
8. Log to results.tsv
9. Update resume.md
10. Continue
```

## Retry Protocol

When an iteration fails:

```
Attempt 1: Fix the specific error
Attempt 2: Try simpler implementation
Attempt 3: Decompose into smaller change

After 3 failures:
- Log failure with evidence
- Discard the approach
- Try different hypothesis
- If 3 consecutive hypothesis failures, ask user
```

## Change Feedback Format

Use priority markers in logs:

```
✅ KEEP: [feature/fix] — build passes, tests pass
❌ DISCARD: [feature/fix] — [what broke]
🔄 RETRY (N/3): [feature/fix] — [specific error]
⚠️ ESCALATE: [feature/fix] — needs user decision on [X]
```

## Example Session

```text
goal: Implement user authentication feature
eval_command: npm test && npm run build && echo "exit_code=$?"
metric_key: exit_code
metric_direction: lower-is-better (0 = success)
success_criteria: exit_code=0 AND login flow works

Iteration 0: baseline - build passes, no auth
Iteration 1: ✅ KEEP add auth types and interfaces
Iteration 2: ✅ KEEP add auth context provider
Iteration 3: 🔄 RETRY add login form — type error
Iteration 3b: ✅ KEEP add login form (fixed types)
Iteration 4: ✅ KEEP add protected route HOC
Iteration 5: ✅ KEEP add logout functionality
Iteration 6: ✅ KEEP add auth persistence
Goal met: auth feature complete, all tests pass
```

## Framework-Specific Notes

### React/Next.js
```bash
# Dev verification
npm run dev  # check no console errors

# Production verification
npm run build && npm run start

# Common issues
- Hydration mismatch → ensure server/client consistency
- Missing keys → add unique keys to list items
- useEffect deps → include all dependencies
```

### Python/FastAPI/Django
```bash
# Verify
uv run pytest
uv run python -m mypy src/

# Django specific
uv run python manage.py check
uv run python manage.py migrate --check
```

## Handoff Template

When pausing or completing:

```markdown
## App Loop Handoff

**Feature Status:**
- Implemented: [what's done]
- Remaining: [what's left]
- Blockers: [any issues]

**Quality:**
- Build: ✅/❌
- Tests: X/Y passing
- Types: ✅/❌

**Files Modified:**
- src/components/Auth.tsx — new component
- src/context/AuthContext.tsx — new context

**Commits:**
- abc1234: add auth types
- def5678: add auth context

**Next Steps:**
1. [what to do next]
```

## Common Pitfalls

- **Breaking the build**: Always verify build before keeping
- **Untested features**: Add tests for new functionality
- **Ignoring type errors**: Fix TypeScript errors, don't suppress
- **Big bang changes**: Small iterations, not massive rewrites
- **Skipping phases**: Foundation before UI, always
