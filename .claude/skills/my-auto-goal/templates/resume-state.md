# Resume State Template

Template for `resume.md` — enables context recovery after compaction. Overwrite completely after EVERY iteration. Never append.

If context is lost, read this file FIRST, then `results.tsv`, then `session.md`.

---

```markdown
# Resume State

**Last Updated:** <YYYY-MM-DD HH:MM:SS>
**Session ID:** <YYYYMMDD-HHMMSS>

## Position

```yaml
iteration: <N>
best_score: <score>
best_iteration: <N>
current_score: <score>
no_improve_count: <N>
```

## Locks

```yaml
domain: <research | code | debug | app | content | web-seo>
scope: <path>
project_root: <path>
soft_pivot: <N>
max_no_improve: <N>
```

## Last Iteration

```yaml
change: <brief description>
result: <keep | discard | crash | violation | timeout>
delta: <+X | -X | 0>
```

## Next

```yaml
hypothesis: <what to try next>
blocked: <issue or "none">
```

## Recovery

You are the autogoal lead for session `<session_id>`.
1. Read `session.md` for full goal spec
2. Read `results.tsv` for iteration history
3. Continue from iteration `<N + 1>`
4. Do NOT re-run baseline
5. Next hypothesis: `<hypothesis>`

| Field | Value |
|-------|-------|
| Goal | <brief goal> |
| Metric | <metric_key> (<direction>) |
| Success | <success_criteria> |
```
