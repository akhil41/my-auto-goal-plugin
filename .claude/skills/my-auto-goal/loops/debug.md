# Debug Domain Loop

Guidance for `domain_type: debug` loops.

## Scope

Find and fix ONE specific bug. No feature additions, no unrelated refactoring.

## Key Difference from Code Loop

Code loop is metric-driven (improve a score). Debug loop is outcome-driven (bug gone or not). Uses `bug_status`: 0 (reproduces), 50 (partial fix), 100 (fixed + regression test).

## Debug Pattern (5-Step)

```
1. REPRODUCE — Document exact steps, capture evidence (logs, errors, stack trace)
2. ISOLATE   — Narrow to specific file/function/line, add debug logging
3. HYPOTHESIZE — Form ONE specific root cause hypothesis
4. FIX       — Implement minimal fix. Wrong hypothesis → discard, new hypothesis
5. VERIFY    — Bug gone + all tests pass + regression test added + debug logging removed
```

## Quality Gate

| Check | Pass criteria |
|-------|--------------|
| Bug no longer reproduces | Run repro steps → passes |
| No regressions | Full test suite passes |
| Root cause addressed | Not a workaround |
| Regression test added | New test exists |

## Iteration Pattern

1. Form root cause hypothesis
2. Implement fix attempt
3. Test reproduction steps
4. Decision:
   - Fixed + tests pass → KEEP (commit)
   - Still exists → DISCARD (revert), new hypothesis
   - Fixed but regressions → DISCARD (revert), refine fix
5. Log with hypothesis tracking:
```
✅ FIX VERIFIED: [root cause] → [fix applied]
❌ HYPOTHESIS WRONG: [hypothesis] — evidence: [why wrong] → next: [new hypothesis]
```

## Retry Protocol

After 5 failed hypotheses: step back, add more logging, consider asking user for context. Check if bug is consistently reproducible.

## Git Workflow

```bash
git checkout -b fix/issue-N-description

# Wrong hypothesis → revert
git restore . && git clean -fd

# Correct → commit fix + test separately
git add <fix-files>
git commit -m "fix(module): description\n\nRoot cause: ...\nFixes #N"
git add <test-files>
git commit -m "test(module): regression test for #N"
```
