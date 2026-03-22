---
name: my-auto-goal
description: Use when the user wants an autonomous goal loop that keeps iterating inside the current project directory, with upfront approval, locked scope, and minimal interruptions during research, coding, debugging, app work, SEO work, or content iteration.
---

# my-auto-goal

> *Give it a goal. It keeps iterating until it gets there or cleanly pauses.*

Autonomous goal-achievement loop inspired by karpathy/autoresearch, generalized for research, code, debug, app, content, and web/SEO work.

## Trigger

This skill is intended for explicit `/my-auto-goal` use.
Do not rely on automatic keyword triggering.

## Core Design

The **main session** (the one invoking this skill) is the lead. It:
- Clarifies the goal with the user
- Presents the plan and gets approval
- Owns the loop, strategy, scoring, and all keep/discard decisions
- Dispatches subagents for bounded tasks (especially web research)

**Subagents** are bounded workers dispatched via the Agent tool:
- Use `subagent_type: "Explore"` or `subagent_type: "general-purpose"` for web research tasks
- The subagent prompt should include instructions for web search
- Generic subagents for other bounded tasks

**Important:**
- The lead (main session) owns all decisions
- Subagents return results; the lead decides keep/discard
- Do not allow work outside the directory where the skill started
- Do not let subagents improvise outside their assigned tasks

## Web Research

For web research tasks, dispatch a general-purpose subagent with explicit web search instructions:

```
Agent({
  subagent_type: "general-purpose",
  prompt: `
    You are performing web research for an autogoal loop.

    Session directory: <session_dir>

    Research task: [specific question]

    Instructions:
    1. Use WebSearch to find relevant information
    2. Use WebFetch to extract content from top results
    3. Write detailed findings to: <session_dir>/domain/research/iter-<N>/findings.md
    4. Return a concise summary of key findings

    Constraints:
    - Max 5 searches, max 5 URL fetches
    - Stay focused on the specific question
    - Do not make implementation decisions
  `,
  description: "Web research for [topic]"
})
```

## Non-Negotiable Rules

- **Project boundary is locked to the starting directory.** The loop may use `<project_root>` or a narrower subdirectory inside it. Never go outside it.
- **Domain is locked at startup.** If the loop starts as `research`, it stays `research`. If another domain is required later, finish the current iteration, write a handoff note, and stop.
- **The lead (main session) owns all decisions.** Subagents may gather, run, or summarize, but only the lead decides keep/discard, installs, or scope changes.
- **Use project-local tooling only.** For Python use `uv` only. Never use pip directly. Never install into system Python. For JS use the existing project package manager without global flags.
- **Everything created by this skill lives inside the project.** Never write outside the project root.

## Prerequisites

Check only what is needed for the selected domain.

### All domains
- writable project directory

### code / app / debug / web-seo
- git repo exists: `git rev-parse --git-dir`
- working tree is clean before loop starts: `git diff --quiet && git diff --cached --quiet`
- project has a working evaluation command (or reproduction steps for debug)
- for Python projects: `uv` available (`uv --version`), or fallback to existing venv if uv not installed

### research / content
- git is optional
- no install is required unless the user explicitly approves one during setup

If a prerequisite is missing, stop and tell the user exactly what is missing and how to fix it.

## Phase 1: Goal Clarification

Ask questions until this goal spec is complete.
Prefer one question at a time, but batch 2-3 tightly related yes/no or multiple-choice fields when that clearly reduces friction.

```text
goal: <specific objective>
success_criteria: <how to know it is done>
constraints: <must not change / boundaries>
domain_type: <research | code | debug | app | content | web-seo>
project_root: <default = current working directory>
scope: <default = same as project_root, or narrower inside it>
eval_command: <required for code/app/web-seo; reproduction steps for debug>
metric_key: <required for code/app/web-seo; bug_status for debug; rubric_score for research/content>
metric_direction: <higher-is-better | lower-is-better>
install_policy: <none | small-upfront | medium-upfront>
iteration_timeout: <default = 5 minutes for code, 10 minutes for research/content>
soft_pivot: <default 5>
max_no_improve: <default 10>
```

### Install policy definitions
- `none` — zero new dependencies allowed
- `small-upfront` — at most 3 project-local packages, ~50MB total, listed explicitly before launch
- `medium-upfront` — at most 10 project-local packages, ~200MB total, listed explicitly before launch

### Success criteria guidance
- **research:** name a target score or explicit completion condition, e.g. "done when rubric_score >= 85 and all browser families are covered"
- **code:** include an objective metric, e.g. "runtime down by 30% while all tests pass"
- **debug:** bug no longer reproduces + regression test added, e.g. "bug_status = 100"
- **app:** include build/test/acceptance behavior, e.g. "feature works and project builds cleanly"
- **content:** include rubric target or concrete acceptance rule
- **web-seo:** include primary metric threshold, e.g. "Lighthouse SEO >= 95"

### Domain lock guidance
- `research`: gather, compare, synthesize, and write findings. Running existing read-only project scripts is allowed; writing or modifying code is not.
- `code`: modify source and run evals. No drift into broader product research.
- `debug`: find and fix a specific bug. No feature additions or unrelated refactoring.
- `app`: implement app behavior and verify with build/test. No unrelated research loop.
- `content`: write and revise content only.
- `web-seo`: audit and improve web/SEO metrics only.

If the user asks for a mixed goal, decompose it. Start one loop per domain.

## Metric Extraction Contract

Before starting the loop, verify that the metric can be extracted from eval output.

### For code / app / web-seo domains

Accept ONE of these formats:

**Text metric:** Eval command output contains a line matching:
```
<metric_key>=<number>
```
Example: if `metric_key` is `test_coverage`, look for `test_coverage=87.5`

**JSON metric:** Eval command writes JSON to stdout or a file, and `metric_key` is a dot-path:
```
metric_key: results.coverage.percentage
```
Parse JSON and extract the value at that path.

**Exit code metric:** If `metric_key` is `exit_code`, use the command's exit code (0 = success).

### For debug domains

Use **bug_status** outcome-based scoring:

```text
bug_status values:
- 0 = bug still reproduces
- 50 = partially fixed (some cases work)
- 100 = fully fixed + regression test added

success_criteria: bug_status = 100
```

The lead tests reproduction steps after each fix attempt and scores accordingly.

### For research / content domains

Use **rubric-based self-scoring**. The lead evaluates each iteration against a rubric.

**Research rubric** (see `loops/research.md` for details):
```text
Completeness (0-25) + Accuracy (0-25) + Clarity (0-25) + Actionability (0-25) = 0-100
```

**Content rubric** (see `loops/content.md` for details):
```text
Completeness (0-25) + Clarity (0-25) + Engagement (0-25) + Accuracy (0-25) = 0-100
```

After each iteration, the lead self-scores and logs the rubric_score. This is subjective but consistent within a session.

### Baseline failure handling

If iteration 0 (baseline) fails to produce a parseable metric:
1. Log the failure with the raw output
2. Ask the user: "Baseline eval failed. Output: [truncated]. Fix eval_command or provide expected format?"
3. Do not proceed until baseline produces a valid score

## Phase 2: Plan Brief & One Approval

Present this exact kind of brief before launching anything:

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Goal:        <goal>
Domain:      <domain_type> (LOCKED)
Project:     <project_root> (LOCKED)
Scope:       <scope> (LOCKED)
Lead:        this session (main)
Eval:        <eval_command or "rubric-based self-scoring">
Metric:      <metric_key> (<direction>)
Installs:    <install_policy>
Timeout:     <iteration_timeout> per iteration
Soft pivot:  <soft_pivot>
Hard pause:  <max_no_improve>
Will stop if:<goal met | stagnation | scope boundary | user stop>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Approval needed once now:
- create <project_root>/.autogoal-<YYYYMMDD-HHMMSS>/
- run autonomous loop in this session
- dispatch subagents for web research tasks
- create git branch for code/app/web-seo domains
- keep all work inside the locked project/scope
Proceed? (yes / modify first)
```

Do not start the loop until the user approves.

## Phase 3: Session Setup

Once approved:

1. Set `PROJECT_ROOT` to the directory where the skill was started.
2. Reject any requested scope outside `PROJECT_ROOT`.
3. Create a timestamped session directory inside the project (use `YYYYMMDD-HHMMSS` format to avoid collisions):

```text
<project_root>/.autogoal-<YYYYMMDD-HHMMSS>/
├── session.md       # goal spec, lock rules, session config
├── results.tsv      # iteration log
├── resume.md        # recovery state
├── tasks/           # task breakdown (optional)
├── logs/            # verbose logs
├── outputs/         # iteration artifacts
└── domain/          # domain-specific work (e.g., domain/research/iter-1/)
```

4. Initialize `results.tsv` with:

```text
iter	id	score	delta	status	description
```

Where `id` is:
- git commit hash (short) for code/app/web-seo with commits
- `iter-<N>` for research/content or when no commit made

5. Run prerequisite verification commands.
6. For code/app/web-seo loops, if the repo is on `main`/`master` or another shared branch, create a dedicated branch:
   ```bash
   git checkout -b autogoal/<YYYYMMDD-HHMMSS>
   ```
7. Write `session.md` with goal spec, lock rules, session file paths.
8. Write `resume.md` with current iteration = 0.

## Phase 4: The Loop

The main session runs the loop directly. Iteration 0 is always baseline.

```text
1. BASELINE (iter 0)
   - run eval_command with no changes (or self-score current state for research/content)
   - capture the baseline score
   - if baseline fails, stop and ask user to fix
   - log baseline to results.tsv

2. ITERATE (iter >= 1)
   - choose one hypothesis (one focused change)
   - if web research needed, dispatch general-purpose subagent with web instructions
   - if other bounded work needed, dispatch appropriate subagent
   - implement or synthesize based on findings
   - evaluate (run eval_command or self-score)
   - compute delta (see direction-aware formula below)
   - apply keep/discard rules (see below)
   - LOG every iteration to results.tsv
   - WRITE resume.md after every iteration
   - check stopping conditions
   - continue if not stopped
```

### Delta Calculation (Direction-Aware)

**For higher-is-better metrics:**
```
delta = current_score - previous_best
improved = delta > 0
```

**For lower-is-better metrics:**
```
delta = previous_best - current_score
improved = delta > 0
```

In both cases, `delta > 0` means improvement. This ensures keep/discard logic works correctly regardless of metric direction.

### Keep/Discard Rules

**KEEP** if:
- `delta > 0` (improved), OR
- `delta == 0` AND the change reduces complexity (fewer lines, fewer dependencies, simpler logic)

**DISCARD** if:
- `delta < 0` (worse)
- For code/app/debug/web-seo: `git restore . && git clean -fd` to revert changes and remove untracked files
- For research/content: delete the iteration's output files

### No-Improve Definition

An iteration counts as "no-improve" if:
- `delta <= 0` (score did not improve, using direction-aware delta)

The `no_improve_count` increments for each no-improve iteration and resets to 0 when an iteration improves.

### Iteration Timeout

Each iteration has a timeout (default 5 min for code, 10 min for research). If exceeded:
1. Mark iteration as `timeout` in results.tsv
2. Discard any partial work
3. Log timeout reason
4. Continue to next iteration (counts as no-improve)

## Stopping Conditions

### Goal met
When `current_score` meets `success_criteria` (direction-aware):
- For higher-is-better: `current_score >= target`
- For lower-is-better: `current_score <= target`
- For debug (bug_status): `bug_status == 100`
- Notify the user
- Write final report
- Stop the loop

### Stagnation - Soft Pivot
After `soft_pivot` consecutive no-improve iterations:
- Log: "Soft pivot triggered. Trying meaningfully different approach."
- "Meaningfully different" means: change a different aspect of the system, try a different algorithm, or target a different sub-metric
- Do NOT reset `no_improve_count` — soft pivot is an intervention, not a reset

### Stagnation - Hard Pause
After `max_no_improve` consecutive no-improve iterations:
- Pause the loop
- Ask user: "No improvement after N iterations. Continue with new direction, or stop?"
- If continue: reset `no_improve_count` to 0 and get new hypothesis from user
- If stop: write final report and exit

### Scope boundary reached
If the locked domain or locked scope is no longer sufficient:
- Finish the current iteration
- Write handoff note to `session.md`:

```text
## Boundary Handoff
Reason: <why current loop cannot continue safely>
Recommended next domain: <research | code | debug | app | content | web-seo>
Recommended next scope: <path(s) inside project_root>
Artifacts to carry forward:
- <file>
Blocked on:
- <install / scope / domain issue>
Next-loop suggestion:
- <one sentence>
```

- Stop and ask the user to start a fresh loop

### User stop
Write final report and stop cleanly.

## Scope Verification (Mandatory)

Before keeping ANY iteration, verify:
1. All touched paths are inside `<project_root>` and inside locked `scope`
2. No path contains `..` traversal
3. No symlink escapes project root
4. Work stayed inside locked domain (research didn't modify code, code didn't do unscoped research)

If ANY check fails:
- Mark iteration as `violation` in results.tsv
- Record reason in session.md
- Discard the output
- Continue to next iteration (counts as no-improve)

## Context Recovery

After every iteration, overwrite `resume.md` with:

```markdown
# Resume State

iteration: <N>
best_score: <score>
best_iteration: <N>
current_score: <score>
no_improve_count: <N>
soft_pivot: <threshold>
max_no_improve: <threshold>
domain_lock: <domain>
scope_lock: <path>
last_change: <brief description>
last_result: <keep | discard | crash | violation | timeout>
next_hypothesis: <what to try next>
pending_issue: <any boundary issue, or "none">
```

If context is compacted or lost:
1. Read `resume.md`
2. Read `results.tsv`
3. Read `session.md`
4. Continue from `iteration + 1` without re-running baseline

## Final Report

Use this structure:

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AUTOGOAL SESSION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Goal:          <goal>
Domain:        <domain_type>
Project root:  <project_root>
Total iters:   <N>
Best score:    <score> (iter <N>)
Improvement:   <delta from baseline> (<percentage>%)
Status:        <goal met | stagnated | scope boundary | user stopped>

Top improvements:
1. iter N — <description> (+X)
2. iter N — <description> (+X)
3. iter N — <description> (+X)

Best artifact: <file / commit / output>
Full log:      <project_root>/.autogoal-<timestamp>/results.tsv
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Domain-Specific Guides

For detailed guidance on each domain, see:

| Domain | Guide |
|--------|-------|
| `code` | `loops/code.md` — quality gates, git workflow, retry protocol |
| `debug` | `loops/debug.md` — reproduce-isolate-fix pattern, hypothesis tracking |
| `app` | `loops/app.md` — feature implementation, framework-specific notes |
| `web-seo` | `loops/web-seo.md` — Lighthouse integration, priority order |
| `research` | `loops/research.md` — source evaluation, subagent dispatch |
| `content` | `loops/content.md` — revision strategies, version control |

## Templates

| Template | Location | Purpose |
|----------|----------|---------|
| Session file | `templates/goal-session.md` | Structure for `session.md` |
| Resume state | `templates/resume-state.md` | Structure for `resume.md` |

## Alignment with karpathy/autoresearch

This skill keeps the core autoresearch pattern:
- baseline first
- one focused change per iteration
- objective evaluation (or consistent self-scoring for research/content)
- keep/discard discipline
- autonomous looping
- time-bounded iterations

It extends autoresearch with:
- domain lock (prevents scope creep)
- project-root lock (security boundary)
- resumable session state (survives context compaction)
- web research via subagents
- concrete install discipline
- explicit metric extraction contract
- rubric-based scoring for non-code domains
- quality gates with retry protocol (from agency-agents patterns)
- structured handoff templates
