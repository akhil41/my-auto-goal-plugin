# Research Domain Loop

Guidance for `domain_type: research` loops.

## Scope

Gather, compare, synthesize, and write findings. **No code modifications allowed.**

Allowed: web searches, URL fetches, reading project files, running read-only scripts, writing to session directory.

## Parallel Subagent Dispatch

Research loops should maximize parallelism. Dispatch **multiple subagents simultaneously**:

```
# Launch 2-3 research subagents in a single message
Agent({
  subagent_type: "general-purpose",
  run_in_background: true,
  prompt: "Search for [aspect 1]. Use WebSearch and WebFetch. Write to <session>/domain/research/iter-N/aspect-1.md. Return summary.",
  description: "Research: [aspect 1]"
})
Agent({
  subagent_type: "general-purpose",
  run_in_background: true,
  prompt: "Search for [aspect 2]. Use WebSearch and WebFetch. Write to <session>/domain/research/iter-N/aspect-2.md. Return summary.",
  description: "Research: [aspect 2]"
})
```

Each subagent: max 5 searches, max 5 URL fetches. Stay focused on assigned question.

## Rubric (Self-Scoring)

| Dimension | 0-25 | Criteria |
|-----------|------|----------|
| Completeness | 0-25 | Covers all required aspects? |
| Accuracy | 0-25 | Claims supported by sources? |
| Clarity | 0-25 | Well-organized and readable? |
| Actionability | 0-25 | Someone can act on this? |

Score consistently. Document reasoning. Compare to previous iteration.

## Source Quality

| Quality | Use |
|---------|-----|
| High (official docs, primary sources) | Cite directly |
| Medium (reputable blogs, SO answers) | Verify claims |
| Low (forums, outdated) | Cross-reference only |
| Avoid (AI summaries, content farms) | Skip |

## Iteration Pattern

1. Identify gap or improvement area in current findings
2. Dispatch parallel subagents for web research
3. Synthesize findings into structured output
4. Self-score → keep if improved, discard if not
5. Log `✅ KEEP: [change] Rubric: X→Y (+Z)` or `❌ DISCARD: [change] Rubric: X→Y (-Z)`

## Output Structure

```markdown
# [Research Topic]
## Executive Summary
## Key Findings
### Finding 1: [Title]
- Source: [URL] | Confidence: High/Medium/Low
## Comparison Table
## Recommendations
## Sources
## Gaps & Uncertainties
```
