# Resume State Template

Template for `resume.md` — enables context recovery after compaction.

---

```markdown
# Resume State

**Last Updated:** <YYYY-MM-DD HH:MM:SS>
**Session ID:** <YYYYMMDD-HHMMSS>

## Current Position

```yaml
iteration: <N>
best_score: <score>
best_iteration: <N>
current_score: <score>
no_improve_count: <N>
```

## Thresholds

```yaml
soft_pivot: <N>
max_no_improve: <N>
iteration_timeout: <minutes>
```

## Locks

```yaml
domain_lock: <research | code | debug | app | content | web-seo>
scope_lock: <path>
project_root: <path>
```

## Last Iteration

```yaml
last_change: <brief description of what was tried>
last_result: <keep | discard | crash | violation | timeout>
last_delta: <+X | -X | 0>
```

## Next Steps

```yaml
next_hypothesis: <what to try next>
pending_issue: <any boundary issue, or "none">
```

## Recovery Instructions

If you are reading this after context compaction:

1. You are the autogoal lead for session `<session_id>`
2. Read `session.md` for full goal specification
3. Read `results.tsv` for iteration history
4. Continue from iteration `<N + 1>`
5. Do NOT re-run baseline (iter 0)
6. Your next hypothesis is: `<next_hypothesis>`

## Quick Reference

| Metric | Value |
|--------|-------|
| Goal | <brief goal> |
| Metric | <metric_key> (<direction>) |
| Success | <success_criteria> |
| Domain | <domain> (LOCKED) |
```

---

## Usage

1. Create this file at `<session_dir>/resume.md` during Phase 3 (Session Setup)
2. Overwrite completely after EVERY iteration
3. If context is lost, read this file FIRST to recover state
4. Never append — always overwrite with current state
