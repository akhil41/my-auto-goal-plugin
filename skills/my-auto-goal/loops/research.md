# Research Domain Loop

Domain-specific guidance for `domain_type: research` loops.

## Scope

Gather, compare, synthesize, and write findings on a topic.

**Allowed:**
- Web searches and URL fetches
- Reading existing project files
- Running read-only project scripts
- Writing research outputs to session directory
- Synthesizing information from multiple sources

**Not allowed:**
- Writing or modifying code
- Installing packages
- Changing project configuration
- Actions that modify project state

## Quality Gate (Self-Scoring Rubric)

After each iteration, self-score against this rubric:

| Dimension | 0-25 Scale | Criteria |
|-----------|------------|----------|
| **Completeness** | 0-25 | Does it cover all required aspects? |
| **Accuracy** | 0-25 | Are claims supported by sources? |
| **Clarity** | 0-25 | Is it well-organized and readable? |
| **Actionability** | 0-25 | Can someone act on this information? |

**rubric_score = sum of dimensions (0-100)**

**Scoring discipline:**
- Be consistent across iterations (same standard)
- Document reasoning for each score
- Compare to previous iteration explicitly

## Research Pattern

```
1. Define research question (from goal)
2. Identify 3-5 search queries
3. Execute searches (via subagent)
4. Evaluate source quality
5. Extract key findings
6. Synthesize into structured output
7. Self-score with rubric
8. Compare to previous best
9. Keep if improved, refine if not
```

## Source Evaluation

Rate each source before using:

| Quality | Indicators | Use? |
|---------|-----------|------|
| **High** | Official docs, peer-reviewed, primary source | ✅ Cite directly |
| **Medium** | Reputable blogs, tutorials, Stack Overflow answers | ✅ Verify claims |
| **Low** | Random forums, outdated content, unverified | ⚠️ Cross-reference only |
| **Avoid** | AI-generated summaries, content farms | ❌ Don't use |

## Iteration Pattern

```
1. Read current findings state
2. Identify gap or improvement area
3. Formulate specific research question
4. Dispatch subagent for web research
5. Integrate findings into output
6. Self-score with rubric
7. Decision:
   - Score improved → KEEP (overwrite output)
   - Score unchanged → KEEP if simpler/clearer
   - Score dropped → DISCARD (revert output)
8. Log to results.tsv
9. Update resume.md
10. Continue until goal met
```

## Output Structure

Research outputs should follow this structure:

```markdown
# [Research Topic]

## Executive Summary
[2-3 sentence overview of key findings]

## Key Findings

### Finding 1: [Title]
[Description with evidence]
- Source: [URL]
- Confidence: High/Medium/Low

### Finding 2: [Title]
...

## Comparison Table
| Option | Pros | Cons | Best For |
|--------|------|------|----------|
| A | ... | ... | ... |
| B | ... | ... | ... |

## Recommendations
1. [Actionable recommendation]
2. [Actionable recommendation]

## Sources
1. [URL] - [what it provided]
2. [URL] - [what it provided]

## Gaps & Uncertainties
- [What we don't know yet]
- [Conflicting information found]
```

## Subagent Dispatch

For web research, dispatch with clear instructions:

```
Agent({
  subagent_type: "general-purpose",
  prompt: `
    Research question: [specific question]

    Context: [what we're trying to learn]

    Instructions:
    1. Search for: [specific queries]
    2. Prioritize: [official docs, recent sources, etc.]
    3. Extract: [specific data points needed]
    4. Write findings to: <session_dir>/domain/research/iter-<N>/

    Return: Summary of key findings with source URLs
  `,
  description: "Research: [topic]"
})
```

## Change Feedback Format

```
✅ KEEP: expanded section on [topic]
   Rubric: 65→72 (+7)
   - Completeness: 15→18 (added missing aspect)
   - Accuracy: 20→20 (maintained)
   - Clarity: 15→17 (better structure)
   - Actionability: 15→17 (clearer recommendations)

❌ DISCARD: rewrote comparison table
   Rubric: 72→68 (-4)
   - Clarity dropped (table too complex)

🔄 CONTINUE: need more sources on [aspect]
```

## Example Session

```text
goal: Research best practices for API rate limiting
success_criteria: rubric_score >= 85 with all major approaches covered
metric_key: rubric_score
metric_direction: higher-is-better

Iteration 0: baseline outline = 30
  Completeness=5, Accuracy=10, Clarity=10, Actionability=5

Iteration 1: ✅ KEEP add token bucket algorithm section
  Rubric: 30→48 (+18)

Iteration 2: ✅ KEEP add sliding window section
  Rubric: 48→62 (+14)

Iteration 3: ✅ KEEP add comparison table
  Rubric: 62→75 (+13)

Iteration 4: ✅ KEEP add implementation examples
  Rubric: 75→86 (+11)

Goal met: 86 >= 85
```

## Handoff Template

```markdown
## Research Loop Handoff

**Topic:** [research topic]

**Final Score:** X/100
- Completeness: X/25
- Accuracy: X/25
- Clarity: X/25
- Actionability: X/25

**Output Location:**
<session_dir>/outputs/research-final.md

**Coverage:**
- ✅ [aspect covered]
- ✅ [aspect covered]
- ⚠️ [aspect partially covered]
- ❌ [aspect not covered]

**Source Quality:**
- X high-quality sources
- Y medium-quality sources
- Z sources cross-referenced

**Gaps:**
- [what wasn't found]
- [conflicting information]

**Recommendations for Follow-up:**
1. [further research needed]
```

## Common Pitfalls

- **Source quality blindness**: Always evaluate source credibility
- **Recency bias**: Check publication dates, prefer recent for tech topics
- **Confirmation bias**: Seek disconfirming evidence too
- **Over-collection**: Synthesize as you go, don't just accumulate
- **Missing citations**: Always track source URLs
- **Scope creep**: Stay focused on the research question
