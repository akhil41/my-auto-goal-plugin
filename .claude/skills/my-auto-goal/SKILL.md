---
name: my-auto-goal
description: Use when the user wants an autonomous goal loop that keeps iterating inside the current project directory, with upfront approval, locked scope, and minimal interruptions during research, coding, debugging, app work, SEO work, or content iteration. Trigger on phrases like "auto goal", "keep iterating", "autonomous loop", "research loop", "improve until", or any request for repeated autonomous improvement cycles.
---

# my-auto-goal

> *Give it a goal. It keeps iterating until it gets there or cleanly pauses.*

Autonomous goal-achievement loop inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch), generalized for research, code, debug, app, content, and web/SEO work.

**CRITICAL: Do NOT invoke `Skill(my-auto-goal-plugin:my-auto-goal)` from within this skill. You are already inside it. Doing so creates an infinite loop.**

## Execution Order

You MUST execute these phases in strict order. Do NOT skip to Phase 4 (the loop). Each phase has a **gate** — you cannot proceed past it until it is satisfied.

```
Phase 1: Goal Clarification  → GATE: all required fields have values
Phase 2: Plan Brief & Approval → GATE: user said "yes" or "proceed"
Phase 3: Session Setup         → GATE: session directory + files exist on disk
Phase 4: The Loop              → only after all 3 gates passed
```

## Core Design

The **main session** (lead) owns the loop: clarifies the goal, gets approval, dispatches subagents, scores iterations, and makes all keep/discard decisions.

**Subagents** are bounded workers. Dispatch them via the Agent tool for:
- Web research (`subagent_type: "general-purpose"`)
- Code exploration (`subagent_type: "Explore"`)
- Any bounded parallel task

Always dispatch **multiple subagents in parallel** when possible — use `run_in_background: true` for concurrent work.

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
- Never install packages, dependencies, or tools without explicit user approval

## Phase 1: Goal Clarification

**MANDATORY.** Use AskUserQuestion to gather the goal spec. Infer what you can from context, ask about the rest.

Required fields (all must have values before proceeding):
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
iteration_timeout: <default 5min code, 10min research/content>
soft_pivot: <default 5>
max_no_improve: <default 10>
```

**Smart defaults:** For research/content, default to rubric-based scoring. For debug, default to bug_status metric.

### Prerequisite Verification

**Before proceeding to Phase 2, verify ALL prerequisites. Do NOT skip this.**

| Domain | Check | How to verify |
|--------|-------|---------------|
| All | Writable project directory | `ls -la <project_root>` |
| code/app/debug | Git repo exists | `git -C <project_root> status` |
| code/app/debug | Working tree is clean | `git -C <project_root> diff --quiet` |
| code/app/debug | Eval command works | Run `<eval_command>` and check exit code |
| code/app/debug | Test runner available | `which pytest` or `which npm` etc. |
| web-seo | Lighthouse available | `npx lighthouse --version` |
| research/content | None required | — |

**If a prerequisite fails:**
- Missing git repo → tell user: "Run `git init` in your project first"
- Dirty working tree → tell user: "Commit or stash your changes first"
- Test runner missing → tell user: "Install pytest/jest in your project first. I will not install packages for you."
- Eval command fails → tell user exactly what failed and ask them to fix it

**GATE: All prerequisites pass. All required fields have values. Only then proceed to Phase 2.**

## Phase 2: Plan Brief & Approval

**MANDATORY.** Present this brief and wait for explicit approval. Do NOT start the loop without it.

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

Use AskUserQuestion with options ["Yes, proceed", "Modify first"] to get approval.

**GATE: User approved. Only then proceed to Phase 3.**

## Phase 3: Session Setup

**MANDATORY.** Create these files on disk before starting any iteration.

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
3. For code/app/debug/web-seo: create branch `autogoal/<YYYYMMDD-HHMMSS>`
4. Write `session.md` with the goal spec (see `templates/goal-session.md`)
5. Write `resume.md` with iteration = 0 (see `templates/resume-state.md`)
6. Read the domain guide: `loops/<domain_type>.md`

**GATE: Verify session directory exists with `ls <session_dir>`. Only then proceed to Phase 4.**

## Phase 4: The Loop

Iteration 0 is always baseline. Then iterate.

```
BASELINE (iter 0):
  Run eval / self-score current state → baseline score
  Log to results.tsv: 0\t-\t<score>\t-\tbaseline\tInitial state
  If eval fails → ask user to fix eval_command, do NOT proceed

ITERATE (iter >= 1):
  1. Choose ONE hypothesis (one focused change)
  2. Dispatch subagents in parallel for research/exploration if needed
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
| delta > 0 (improved) | **KEEP** — commit (code domains) or save (research/content) |
| delta == 0 and simpler | **KEEP** |
| delta < 0 (worse) | **DISCARD** — `git restore . && git clean -fd` (code) or delete files (research/content) |

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
