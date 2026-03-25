---
name: my-auto-goal
description: Use when the user wants an autonomous goal loop that keeps iterating inside the current project directory, with upfront approval, locked scope, and minimal interruptions during research, coding, debugging, app work, SEO work, or content iteration. Trigger on phrases like "auto goal", "keep iterating", "autonomous loop", "research loop", "improve until", or any request for repeated autonomous improvement cycles.
---

# my-auto-goal

> *Give it a goal. It keeps iterating until it gets there or cleanly pauses.*

Autonomous goal-achievement loop inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch), generalized for research, code, debug, app, content, and web/SEO work.

## Core Design

The **main session** (lead) owns the loop: clarifies the goal, gets approval, dispatches subagents, scores iterations, and makes all keep/discard decisions.

**Subagents** are bounded workers. Dispatch them via the Agent tool for:
- Web research (`subagent_type: "general-purpose"`)
- Code exploration (`subagent_type: "Explore"`)
- Any bounded parallel task

Always dispatch **multiple subagents in parallel** when possible — use `run_in_background: true` for concurrent work. This is the primary way to maximize throughput.

```
Agent({
  subagent_type: "general-purpose",
  run_in_background: true,
  prompt: "Research [topic]. Use WebSearch and WebFetch. Write findings to <path>. Return summary.",
  description: "Research: [topic]"
})
```

**Non-negotiable rules:**
- Project boundary locked to starting directory — never write outside it
- Domain locked at startup — no switching mid-loop
- Lead owns all decisions — subagents gather, lead decides
- Use project-local tooling only (uv for Python, project package manager for JS)

## Phase 1: Goal Clarification

Use AskUserQuestion to efficiently gather the goal spec. Batch related questions to reduce friction.

Required fields:
```yaml
goal: <specific objective>
success_criteria: <measurable completion condition>
constraints: <boundaries / must-not-change>
domain_type: <research | code | debug | app | content | web-seo>
project_root: <default = cwd>
scope: <default = project_root, or narrower>
eval_command: <for code/app/web-seo; repro steps for debug; "rubric" for research/content>
metric_key: <metric name>
metric_direction: <higher-is-better | lower-is-better>
install_policy: <none | small-upfront | medium-upfront>
iteration_timeout: <default 5min code, 10min research/content>
soft_pivot: <default 5>
max_no_improve: <default 10>
```

**Smart defaults:** Infer as much as possible from the user's initial request. Only ask about fields you cannot infer. For research/content, default to rubric-based scoring. For debug, default to bug_status metric.

### Prerequisites (check per domain)

| Domain | Requirements |
|--------|-------------|
| All | Writable project directory |
| code/app/debug/web-seo | Git repo, clean working tree, eval command |
| research/content | None (git optional) |

If missing, stop and tell the user exactly what to fix.

## Phase 2: Plan Brief & Approval

Present this brief and get one approval before starting:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Goal:        <goal>
Domain:      <domain_type> (LOCKED)
Project:     <project_root> (LOCKED)
Scope:       <scope> (LOCKED)
Eval:        <eval_command or "rubric-based self-scoring">
Metric:      <metric_key> (<direction>)
Timeout:     <iteration_timeout> per iteration
Stagnation:  soft pivot at <N>, hard pause at <N>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Will create: <project_root>/.autogoal-<YYYYMMDD-HHMMSS>/
Will do:     autonomous loop, subagent dispatch, git branch (code domains)
Proceed? (yes / modify first)
```

## Phase 3: Session Setup

Once approved:

1. Create timestamped session directory:
```
<project_root>/.autogoal-<YYYYMMDD-HHMMSS>/
├── session.md       # goal spec + session log
├── results.tsv      # iter, id, score, delta, status, description
├── resume.md        # recovery state (overwritten each iteration)
├── outputs/         # iteration artifacts
└── domain/          # domain-specific work
```

2. Initialize `results.tsv` with header: `iter\tid\tscore\tdelta\tstatus\tdescription`
3. For code/app/web-seo: create branch `autogoal/<YYYYMMDD-HHMMSS>`
4. Write `session.md` from template (see `templates/goal-session.md`)
5. Write `resume.md` with iteration = 0

## Phase 4: The Loop

Iteration 0 is always baseline. Then iterate.

```
BASELINE (iter 0):
  Run eval / self-score current state → baseline score
  If fails, ask user to fix eval_command

ITERATE (iter >= 1):
  1. Choose ONE hypothesis (one focused change)
  2. Dispatch subagents in parallel for research/exploration
  3. Implement or synthesize
  4. Evaluate (run eval or self-score)
  5. Compute delta → apply keep/discard
  6. Log to results.tsv + update resume.md
  7. Check stopping conditions
  8. Continue
```

### Keep/Discard

| Condition | Action |
|-----------|--------|
| delta > 0 (improved) | **KEEP** |
| delta == 0 and simpler | **KEEP** |
| delta < 0 (worse) | **DISCARD** — revert (git restore for code, delete files for research) |

Delta is direction-aware: `delta = current - best` (higher-is-better) or `delta = best - current` (lower-is-better). Positive always means improvement.

### Metric Extraction

| Domain | Method |
|--------|--------|
| code/app/web-seo | Parse `metric_key=<number>` from eval output, or JSON dot-path, or exit code |
| debug | bug_status: 0 (reproduces), 50 (partial), 100 (fixed + test added) |
| research/content | Rubric self-score: Completeness + Accuracy + Clarity + Actionability = 0-100 |

### Stopping Conditions

| Condition | Action |
|-----------|--------|
| Goal met (score meets success_criteria) | Final report → stop |
| Soft pivot (N consecutive no-improve) | Try meaningfully different approach, don't reset counter |
| Hard pause (max_no_improve reached) | Ask user: continue with new direction, or stop? |
| Scope boundary reached | Write handoff note → stop |
| User stop | Final report → stop |

## Scope Verification

Before keeping ANY iteration, verify:
1. All touched paths inside project_root and scope
2. No `..` traversal, no symlink escapes
3. Work stayed inside locked domain

Violation → mark as `violation`, discard, continue.

## Context Recovery

After every iteration, overwrite `resume.md` with current state (see `templates/resume-state.md`).

If context is compacted:
1. Read `resume.md` → `results.tsv` → `session.md`
2. Continue from `iteration + 1` — do NOT re-run baseline

## Final Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AUTOGOAL SESSION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Goal:          <goal>
Domain:        <domain_type>
Total iters:   <N>
Best score:    <score> (iter <N>)
Improvement:   <delta from baseline> (<percentage>%)
Status:        <goal met | stagnated | scope boundary | user stopped>

Top improvements:
1. iter N — <description> (+X)
2. iter N — <description> (+X)

Full log: <session_dir>/results.tsv
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Domain Guides

Read the relevant guide when starting a loop:

| Domain | Guide | Key pattern |
|--------|-------|-------------|
| research | `loops/research.md` | Parallel subagent dispatch, source evaluation, rubric scoring |
| code | `loops/code.md` | Quality gates, git workflow, retry protocol (3 attempts) |
| debug | `loops/debug.md` | Reproduce → isolate → hypothesize → fix → verify |
| app | `loops/app.md` | Foundation → logic → UI → polish phases |
| content | `loops/content.md` | Rubric scoring, revision by weakest dimension |
| web-seo | `loops/web-seo.md` | Lighthouse audit-first, priority-ordered fixes |
