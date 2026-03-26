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

### Subagent Failure Handling

WebSearch and WebFetch may fail due to permissions, model limitations, or blocked domains. Plan for this:

**If subagents return empty/failed results:**
1. Lead performs web research directly using WebSearch and WebFetch (lead has more reliable tool access)
2. If WebSearch fails, try WebFetch on known-good URLs (GitHub repos, official docs, HuggingFace)
3. If WebFetch also fails, use knowledge-based synthesis and clearly mark confidence as "Low — no live web data"

**URLs that tend to work:** GitHub repos, HuggingFace model cards, official documentation sites, Wikipedia.
**URLs that tend to fail:** Reddit, news sites behind paywalls, forums requiring auth, sites with aggressive bot protection.

**Never silently accept empty results.** If a subagent returns nothing useful, acknowledge the gap and try alternative sources.

## Baseline (Iteration 0)

For research, the baseline is "no research exists yet" — score it **0**. This gives a clean starting point. Log: `0\t-\t0\t-\tbaseline\tNo research yet`

## Rubric (Self-Scoring)

| Dimension | 0-5 (Absent) | 6-12 (Weak) | 13-18 (Adequate) | 19-22 (Strong) | 23-25 (Excellent) |
|-----------|-------------|-------------|-------------------|-----------------|-------------------|
| Completeness | No research or major topics missing | Most topics touched but thin | All required aspects covered, some gaps | Thorough coverage with comparisons | Comprehensive — nothing missing, good depth |
| Accuracy | No sources, unsupported claims | Some sources, many unqualified | Mostly sourced, key claims verified | Well-sourced, confidence levels noted | All claims sourced, uncertainties flagged |
| Clarity | Disorganized, hard to parse | Structure exists but rough | Clear sections, some flow issues | Easy to read, good structure and transitions | Executive summary alone is actionable |
| Actionability | Abstract only, no recommendations | Some advice but vague or generic | Concrete recommendations for some scenarios | Clear next steps with trade-offs | Decision-ready with comparison tables, specific guidance |

**Scoring rules:**
- Score each dimension independently using the bands above
- Compare directly to the previous version
- Total = sum of all four dimensions (0-100)
- Record per-dimension scores in session.md iteration log

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

Write research artifacts to `<session_dir>/outputs/`: `research-v1.md`, `research-v2.md`, etc.
- `<session_dir>` = the `.autogoal-<YYYYMMDD-HHMMSS>/` directory created in Phase 3
- **On success:** Copy the best-scoring version to `research-final.md` in the same `outputs/` directory

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
