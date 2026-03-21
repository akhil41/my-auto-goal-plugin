# Code Domain Loop

Domain-specific guidance for `domain_type: code` loops.

## Scope

Modify source code and run evaluations to improve a specific metric.

**Allowed:**
- Edit source files within scope
- Run tests and benchmarks
- Refactor code
- Add/remove code
- Modify configurations within project

**Not allowed:**
- Drift into product research
- Change unrelated files
- Install packages not in approved list
- Modify files outside scope

## Quality Gate

Before advancing from any iteration, verify:

| Check | Command | Pass Criteria |
|-------|---------|---------------|
| Tests pass | `pytest` / `npm test` | Exit code 0 |
| Types check | `mypy` / `tsc --noEmit` | No errors |
| Lint clean | `ruff` / `eslint` | No errors |
| Build works | `uv build` / `npm run build` | Exit code 0 |

**Gate failure handling:**
- FAIL → Retry (max 3 attempts per hypothesis)
- After 3 retries, discard hypothesis and try next
- If 3 consecutive hypotheses fail, escalate to user

## Typical Metrics

| Metric | Direction | Example eval_command |
|--------|-----------|---------------------|
| Test coverage | higher-is-better | `pytest --cov --cov-report=term \| grep TOTAL` |
| Runtime | lower-is-better | `python benchmark.py` |
| Memory usage | lower-is-better | `python -m memory_profiler script.py` |
| Lines of code | lower-is-better | `wc -l src/**/*.py` |
| Cyclomatic complexity | lower-is-better | `radon cc src/ -a -s` |
| Test pass rate | higher-is-better | `pytest --tb=no -q` |

## Hypothesis Generation

For code loops, generate hypotheses by examining:

1. **Performance bottlenecks**
   - Profile the code: `python -m cProfile -s cumtime script.py`
   - Find hot spots and optimize them

2. **Code smells**
   - Long functions → extract methods
   - Duplicate code → refactor to shared utility
   - Deep nesting → flatten with early returns

3. **Algorithm improvements**
   - O(n²) → O(n log n) or O(n)
   - Unnecessary iterations → caching/memoization
   - Blocking I/O → async or batching

4. **Test coverage gaps**
   - Uncovered branches → add tests
   - Edge cases → add boundary tests

## Iteration Pattern

```
1. Read current code state
2. Run baseline eval (iter 0) or current eval
3. Identify ONE specific improvement
4. Implement the change
5. Run quality gate checks
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

When an iteration fails quality gate:

```
Attempt 1: Fix the specific issue
Attempt 2: Try alternative approach
Attempt 3: Simplify the change

After 3 failures:
- Log failure with root cause
- Discard the hypothesis
- Move to next hypothesis
- If 3 consecutive hypothesis failures, escalate to user
```

## Change Feedback Format

When logging iteration results, use markers:

```
✅ KEEP: [description] — metric improved by X%
❌ DISCARD: [description] — metric worsened by X%
🔄 RETRY (N/3): [description] — [specific failure reason]
⚠️ ESCALATE: [description] — [why it needs user input]
```

## Git Workflow

```bash
# Before iteration
git status  # ensure clean

# After successful iteration (KEEP)
git add <changed-files>
git commit -m "autogoal iter N: <description> (+X%)"

# After failed iteration (DISCARD)
git restore . && git clean -fd

# After all iterations
# Lead reviews commits, squashes if needed
```

## Example Session

```text
goal: Reduce test suite runtime by 50%
eval_command: time pytest tests/ 2>&1 | grep real
metric_key: real
metric_direction: lower-is-better
success_criteria: real < 30s (baseline was 60s)

Iteration 0: baseline = 60.2s
Iteration 1: ✅ KEEP parallelize tests with pytest-xdist → 35.1s (-42%)
Iteration 2: ✅ KEEP mock slow external API calls → 28.4s (-19%)
Iteration 3: ✅ KEEP remove redundant setup in fixtures → 25.1s (-12%)
Goal met: 25.1s < 30s (-58% from baseline)
```

## Handoff Template

When pausing or completing, log:

```markdown
## Code Loop Handoff

**Final State:**
- Iterations completed: N
- Best score: X (iter M)
- Improvement from baseline: +/-Y%

**Files Modified:**
- path/to/file.py — [what changed]

**Commits:**
- abc1234: iter 1 — [description]
- def5678: iter 2 — [description]

**Remaining Hypotheses:**
1. [untried approach]
2. [untried approach]

**Blockers/Notes:**
- [any issues encountered]
```

## Common Pitfalls

- **Over-optimizing**: Don't break functionality for marginal gains
- **Premature abstraction**: Don't add complexity to save a few lines
- **Ignoring test failures**: A faster test suite that doesn't pass is worthless
- **Scope creep**: Stay focused on the metric, don't refactor unrelated code
- **Retrying blindly**: If same approach fails twice, try something different
