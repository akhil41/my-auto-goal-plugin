# Debug Domain Loop

Domain-specific guidance for `domain_type: debug` loops.

## Scope

Find and fix a specific bug or unexpected behavior.

**Allowed:**
- Reproducing the bug
- Adding debug logging
- Modifying code to fix the issue
- Adding regression tests
- Removing debug code after fix

**Not allowed:**
- Refactoring unrelated code
- Adding features
- "Improving" code that isn't related to the bug
- Scope creep to other bugs

## Key Difference from Code Loop

| Code Loop | Debug Loop |
|-----------|------------|
| Metric-driven (improve score) | Outcome-driven (bug gone or not) |
| Hypothesis: "this might improve X" | Hypothesis: "this might be the root cause" |
| Success: metric improved | Success: bug no longer reproduces |
| Keep/discard based on delta | Keep/discard based on fix verified |

## Quality Gate

Before keeping any fix:

| Check | Method | Pass Criteria |
|-------|--------|---------------|
| Bug no longer reproduces | Run reproduction steps | Passes |
| No regressions | Run test suite | All pass |
| Root cause addressed | Code review | Not a workaround |
| Tests added | Check for new test | Regression test exists |

**Gate failure = bug still exists or regressions introduced**

## Debug Pattern (5-Step)

```
1. REPRODUCE
   - Document exact reproduction steps
   - Capture evidence (logs, screenshots, error messages)
   - Verify bug exists in current state

2. ISOLATE
   - Narrow down to specific file/function/line
   - Add debug logging if needed
   - Identify the exact failure point

3. HYPOTHESIZE ROOT CAUSE
   - Form ONE specific hypothesis about why
   - "The bug occurs because X happens when Y"
   - Don't guess multiple causes at once

4. FIX
   - Implement minimal fix for the hypothesis
   - If hypothesis wrong → discard, form new hypothesis
   - If fix works → proceed to verify

5. VERIFY
   - Confirm bug no longer reproduces
   - Run full test suite (no regressions)
   - Add regression test if none exists
   - Remove debug logging
```

## Iteration Pattern

```
1. Reproduce bug, capture evidence (iter 0 = baseline)
2. Form root cause hypothesis
3. Implement fix attempt
4. Test reproduction steps
5. Decision:
   - Bug fixed + tests pass → KEEP (commit)
   - Bug still exists → DISCARD (revert, new hypothesis)
   - Bug fixed but regressions → DISCARD (revert, refine fix)
6. Log to results.tsv
7. Update resume.md
8. If not fixed, continue with new hypothesis
```

## Evidence Collection

Always capture before/after evidence:

### Before Fix (Baseline)
```markdown
## Bug Evidence

**Reproduction Steps:**
1. [step 1]
2. [step 2]
3. [step 3]

**Expected:** [what should happen]
**Actual:** [what happens instead]

**Error Message:**
```
[exact error text]
```

**Stack Trace:**
```
[if applicable]
```

**Screenshot/Log:** [path to evidence file]
```

### After Fix (Verification)
```markdown
## Fix Verification

**Same Reproduction Steps:**
1. [step 1]
2. [step 2]
3. [step 3]

**Result:** [expected behavior now occurs]

**Regression Tests:** All pass (X/X)

**New Test Added:** tests/test_bug_123.py
```

## Hypothesis Tracking

Track hypotheses explicitly in results.tsv:

```
iter	hypothesis	result	evidence
0	-	baseline	bug reproduces, error in auth.py:42
1	null user object	WRONG	added null check, still fails
2	race condition in async	WRONG	added lock, still fails
3	stale cache after logout	CORRECT	cleared cache, bug fixed
```

## Retry Protocol

Different from code loop — retries are about finding the RIGHT hypothesis:

```
Hypothesis 1: [guess] → test → wrong
Hypothesis 2: [refined guess based on evidence] → test → wrong
Hypothesis 3: [different angle] → test → correct!

After 5 failed hypotheses:
- Step back and re-examine evidence
- Add more debug logging
- Consider asking user for more context
- Check if bug is actually reproducible consistently
```

## Change Feedback Format

```
✅ FIX VERIFIED: [description]
   Hypothesis: [what was the root cause]
   Fix: [what was changed]
   Evidence: bug no longer reproduces, tests pass
   Regression test: [test file/name]

❌ HYPOTHESIS WRONG: [description]
   Hypothesis: [what we thought]
   Evidence: [why it was wrong]
   Next: [new hypothesis to try]

🔄 PARTIAL FIX: [description]
   Fixed: [what's now working]
   Remaining: [what still fails]
   Next hypothesis: [refined guess]

⚠️ BLOCKED: [description]
   Issue: [why we can't proceed]
   Need: [what information/access is required]
```

## Metric for Debug Loops

Since debug loops are outcome-based, use a simple metric:

```yaml
metric_key: bug_status
metric_direction: higher-is-better

# Values:
# 0 = bug reproduces
# 50 = partially fixed (some cases work)
# 100 = fully fixed + regression test added

success_criteria: bug_status = 100
```

## Example Session

```text
goal: Fix login failing silently when session expires
success_criteria: bug_status = 100 (fixed + regression test)
metric_key: bug_status
metric_direction: higher-is-better

Iteration 0: baseline
  Bug reproduces: user clicks login, nothing happens, no error
  Evidence: console shows 401, but UI doesn't update
  bug_status = 0

Iteration 1: ❌ HYPOTHESIS WRONG
  Hypothesis: API not returning error correctly
  Added logging: API returns 401 correctly
  bug_status = 0

Iteration 2: ❌ HYPOTHESIS WRONG
  Hypothesis: Error handler not catching 401
  Added logging: handler fires, sets error state
  bug_status = 0

Iteration 3: ✅ FIX VERIFIED
  Hypothesis: UI not re-rendering on error state change
  Root cause: stale closure in useEffect
  Fix: added error to dependency array
  Added test: test_login_shows_error_on_401
  bug_status = 100

Goal met: bug fixed + regression test added
```

## Git Workflow for Debug

```bash
# Create debug branch
git checkout -b fix/issue-123-login-silent-fail

# After each hypothesis attempt
# If wrong: revert
git restore . && git clean -fd

# If correct: commit
git add <changed-files>
git commit -m "fix(auth): show error when session expires

Root cause: stale closure in useEffect missing error dependency
Fixes #123"

# Add regression test in separate commit
git add tests/test_auth_error.py
git commit -m "test(auth): add regression test for #123"
```

## Debugging Tools Reference

### Python
```bash
# Add breakpoint
import pdb; pdb.set_trace()
# or
breakpoint()

# Run with verbose logging
python -v script.py
PYTHONVERBOSE=1 python script.py

# Memory debugging
python -m tracemalloc script.py
```

### JavaScript/Node
```javascript
// Add breakpoint
debugger;

// Verbose logging
DEBUG=* node app.js

// Node inspector
node --inspect script.js
```

### General
```bash
# Watch file changes
watch -n 1 'cat /tmp/debug.log | tail -20'

# Capture output
script -q /tmp/session.log
```

## Handoff Template

```markdown
## Debug Loop Handoff

**Bug:** [one-line description]
**Status:** Fixed / Partial / Unresolved

**Root Cause:**
[explanation of why the bug occurred]

**Fix Applied:**
[what was changed and why]

**Files Modified:**
- path/to/file.py — [change description]

**Regression Test:**
- tests/test_bug_fix.py::test_name

**Hypotheses Tried:**
1. [hypothesis] — wrong because [evidence]
2. [hypothesis] — wrong because [evidence]
3. [hypothesis] — CORRECT

**Remaining Issues:**
- [any related issues discovered]

**Commits:**
- abc1234: fix(module): description
- def5678: test(module): regression test
```

## Common Pitfalls

- **Fixing symptoms, not cause**: Ensure you understand WHY, not just WHAT
- **No reproduction steps**: Can't verify fix without consistent reproduction
- **Missing regression test**: Bug will come back without a test
- **Too many changes**: Change ONE thing per hypothesis
- **Ignoring evidence**: Let logs/errors guide you, don't guess blindly
- **Scope creep**: Fix THIS bug, not every bug you notice
