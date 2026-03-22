# Goal Session Template

Template for `session.md` — the persistent record of an autogoal session.

---

```markdown
# Autogoal Session

**Created:** <YYYY-MM-DD HH:MM:SS>
**Session ID:** <YYYYMMDD-HHMMSS>
**Domain:** <research | code | debug | app | content | web-seo>

## Goal Specification

```yaml
goal: <specific objective>
success_criteria: <how to know it is done>
constraints: <must not change / boundaries>
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

| Lock | Value | Set At |
|------|-------|--------|
| Domain | <domain> | Session start |
| Project Root | <path> | Session start |
| Scope | <path> | Session start |

**These locks cannot be changed during the session.**

## Session Files

| File | Purpose |
|------|---------|
| `session.md` | This file — goal spec and session log |
| `results.tsv` | Iteration log with scores |
| `resume.md` | Recovery state for context loss |
| `outputs/` | Iteration artifacts |
| `domain/` | Domain-specific work |
| `logs/` | Verbose logs |

## Iteration Log

<!-- Updated after each iteration -->

| Iter | ID | Score | Delta | Status | Description |
|------|-------|-------|--------|--------|-------------|
| 0 | - | X | - | baseline | Initial state |
| 1 | abc123 | Y | +Z | keep | [what changed] |
| ... | | | | |

## Events

<!-- Log significant events -->

- **<timestamp>**: Session started
- **<timestamp>**: Baseline captured (score=X)
- **<timestamp>**: Soft pivot triggered at iter N
- **<timestamp>**: Goal met / Stagnated / User stopped

## Handoff Notes

<!-- Written when session pauses or completes -->

### Boundary Handoff (if applicable)

**Reason:** <why current loop cannot continue>
**Recommended next domain:** <domain>
**Artifacts to carry forward:**
- <file>

**Blocked on:**
- <issue>

**Next-loop suggestion:**
- <one sentence>

### Completion Notes

**Final state:**
- Best score: X (iter N)
- Improvement: +Y% from baseline
- Status: <goal met | stagnated | user stopped>

**Key changes:**
1. <change> — <impact>
2. <change> — <impact>

**Remaining work:**
- <if any>
```

---

## Usage

1. Create this file at `<session_dir>/session.md` during Phase 3 (Session Setup)
2. Fill in the Goal Specification from the clarified goal
3. Update the Iteration Log after each iteration
4. Add Events as they occur
5. Write Handoff Notes when pausing or completing
