# Goal Session Template

Template for `session.md` — the persistent record of an autogoal session. Create at `<session_dir>/session.md` during Phase 3.

---

```markdown
# Autogoal Session

**Created:** <YYYY-MM-DD HH:MM:SS>
**Session ID:** <YYYYMMDD-HHMMSS>
**Domain:** <research | code | debug | app | content | web-seo>

## Goal Specification

```yaml
goal: <specific objective>
success_criteria: <measurable completion condition>
constraints: <boundaries / must-not-change>
domain_type: <domain>
project_root: <absolute path>
scope: <path or "same as project_root">
eval_command: <command or "rubric-based">
metric_key: <key>
metric_direction: <higher-is-better | lower-is-better>
install_policy: <none | small-upfront | medium-upfront>
iteration_timeout: <minutes>
soft_pivot: <N>
max_no_improve: <N>
```

## Locks

| Lock | Value |
|------|-------|
| Domain | <domain> |
| Project Root | <path> |
| Scope | <path> |

**Locked at session start. Cannot change.**

## Iteration Log

| Iter | Score | Delta | Status | Description |
|------|-------|-------|--------|-------------|
| 0 | X | - | baseline | Initial state |

## Events

- **<timestamp>**: Session started
- **<timestamp>**: Baseline captured (score=X)

## Handoff Notes

### On Completion

**Final state:** best score X (iter N), improvement +Y% from baseline
**Status:** <goal met | stagnated | scope boundary | user stopped>

**Key changes:**
1. <change> — <impact>

**Remaining work:** <if any>
```
