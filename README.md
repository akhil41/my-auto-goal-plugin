# my-auto-goal-plugin

A Claude Code plugin for autonomous goal-achievement loops.

## Inspiration

This plugin is inspired by Andrej Karpathy's [autoresearch](https://github.com/karpathy/autoresearch) - an autonomous research loop that iteratively improves on a research goal. I've generalized the concept beyond research to support six different domains: research, code optimization, debugging, app development, content writing, and web/SEO optimization.

The core idea: **give it a goal, let it iterate until it gets there or cleanly pauses.**

## How It Works

```
┌─────────────────────────────────────────────────────────────┐
│  1. CLARIFY GOAL                                            │
│     Define objective, success criteria, constraints         │
│                           ↓                                 │
│  2. APPROVE PLAN                                            │
│     Review configuration, approve once                      │
│                           ↓                                 │
│  3. AUTONOMOUS LOOP                                         │
│     ┌──────────────────────────────────────┐                │
│     │  Evaluate current state              │                │
│     │           ↓                          │                │
│     │  Generate hypothesis                 │                │
│     │           ↓                          │                │
│     │  Implement change                    │                │
│     │           ↓                          │                │
│     │  Measure improvement                 │                │
│     │           ↓                          │                │
│     │  KEEP (improved) or DISCARD (worse)  │                │
│     │           ↓                          │                │
│     │  Log results, update state           │                │
│     └──────────────────────────────────────┘                │
│                    ↺ repeat until goal met                  │
└─────────────────────────────────────────────────────────────┘
```

## Supported Domains

| Domain | Metric Type | Example Goal |
|--------|-------------|--------------|
| **Research** | Rubric score (0-100) | "Research API rate limiting best practices" |
| **Code** | Objective metric | "Reduce test runtime by 50%" |
| **Debug** | Bug status (0/50/100) | "Fix silent login failure" |
| **App** | Build/test pass | "Implement user authentication" |
| **Content** | Rubric score (0-100) | "Write API documentation" |
| **Web-SEO** | Lighthouse scores | "Achieve SEO score >= 95" |

## Installation

### From GitHub

Add to your `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "my-auto-goal": {
      "source": {
        "source": "github",
        "repo": "akhil41/my-auto-goal-plugin"
      }
    }
  },
  "enabledPlugins": {
    "my-auto-goal-plugin@my-auto-goal": true
  }
}
```

Then restart Claude Code or run `/reload-plugins`.

### Local Installation

```bash
git clone https://github.com/akhilvemuri/my-auto-goal-plugin ~/.claude/plugins/my-auto-goal-plugin
```

Add to settings.json:

```json
{
  "extraKnownMarketplaces": {
    "local": {
      "source": {
        "source": "directory",
        "path": "~/.claude/plugins/my-auto-goal-plugin"
      }
    }
  },
  "enabledPlugins": {
    "my-auto-goal-plugin@local": true
  }
}
```

## Quick Start

```
/my-auto-goal
```

The plugin guides you through goal clarification, then runs autonomously:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Goal:        Reduce test suite runtime by 50%
Domain:      code (LOCKED)
Metric:      runtime (lower-is-better)
Eval:        pytest --durations=0
Will stop:   goal met | stagnation | user stop
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Proceed? (yes / modify first)
```

### Example: Research Loop

```
Goal: Research best practices for API rate limiting
Success: rubric_score >= 85

Iteration 0: baseline = 30
Iteration 1: ✅ KEEP add token bucket section → 48 (+18)
Iteration 2: ✅ KEEP add sliding window section → 62 (+14)
Iteration 3: ✅ KEEP add comparison table → 75 (+13)
Iteration 4: ✅ KEEP add implementation examples → 86 (+11)

Goal met: 86 >= 85
```

### Example: Debug Loop

```
Goal: Fix login failing silently
Success: bug_status = 100

Iteration 0: bug reproduces, bug_status = 0
Iteration 1: ❌ HYPOTHESIS WRONG - API returns 401 correctly
Iteration 2: ❌ HYPOTHESIS WRONG - handler fires correctly
Iteration 3: ✅ FIX VERIFIED - stale closure in useEffect
             Added regression test, bug_status = 100

Goal met!
```

## Key Features

- **Locked boundaries** - Never works outside the starting directory
- **Domain lock** - No mid-session switches between research/code/debug
- **Atomic commits** - Each successful iteration = one commit (code domains)
- **Quality gates** - Tests, types, lint must pass before keeping changes
- **Context recovery** - Can resume after Claude Code compaction via `resume.md`
- **Subagent dispatch** - Web research delegated to focused subagents

## Session Files

Each session creates:

```
.autogoal-YYYYMMDD-HHMMSS/
├── session.md       # Goal spec and session log
├── results.tsv      # Iteration log with scores
├── resume.md        # Recovery state for context loss
├── outputs/         # Iteration artifacts
└── domain/          # Domain-specific work
```

## Configuration

| Option | Default | Description |
|--------|---------|-------------|
| `iteration_timeout` | 5-10 min | Max time per iteration |
| `soft_pivot` | 5 | Iterations before trying new strategy |
| `max_no_improve` | 10 | Max iterations without improvement |
| `install_policy` | none | Package installation policy |

## Acknowledgments

- [Andrej Karpathy](https://github.com/karpathy) for the [autoresearch](https://github.com/karpathy/autoresearch) concept that inspired this plugin
- [Anthropic](https://anthropic.com) for Claude Code and the plugin system

## License

MIT

## Author

Built by [Akhil](https://github.com/akhil41)
