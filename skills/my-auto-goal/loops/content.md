# Content Domain Loop

Domain-specific guidance for `domain_type: content` loops.

## Scope

Write and revise content to meet quality standards.

**Allowed:**
- Writing new content
- Editing existing content
- Restructuring documents
- Improving clarity and flow
- Adding examples and illustrations

**Not allowed:**
- Code changes
- Technical implementation
- Research expansion (that's `research` domain)
- Design changes

## Quality Gate (Self-Scoring Rubric)

After each iteration, self-score against this rubric:

| Dimension | 0-25 Scale | Criteria |
|-----------|------------|----------|
| **Completeness** | 0-25 | Does it cover all required topics? |
| **Clarity** | 0-25 | Is it easy to understand? |
| **Engagement** | 0-25 | Is it interesting to read? |
| **Accuracy** | 0-25 | Are facts correct and claims supported? |

**rubric_score = sum of dimensions (0-100)**

**Scoring discipline:**
- Be consistent across iterations
- Read content aloud to check flow
- Compare directly to previous version

## Content Pattern

```
1. Understand target audience
2. Outline key points
3. Write first draft
4. Self-score
5. Identify weakest dimension
6. Revise to improve that dimension
7. Self-score again
8. Keep if improved, try different approach if not
```

## Iteration Pattern

```
1. Read current content state
2. Self-score with rubric
3. Identify ONE improvement area
4. Make focused revision
5. Re-score
6. Decision:
   - Score improved → KEEP (save new version)
   - Score unchanged → KEEP if clearer/shorter
   - Score dropped → DISCARD (revert)
7. Log to results.tsv
8. Update resume.md
9. Continue until goal met
```

## Revision Strategies by Dimension

### Low Completeness (missing topics)
- Check outline against requirements
- Add missing sections
- Expand thin sections
- Add examples where abstract

### Low Clarity (hard to follow)
- Shorten sentences (max 20 words)
- Use active voice
- Add transitions between paragraphs
- Define jargon on first use
- Use bullet points for lists

### Low Engagement (boring)
- Add a compelling opening hook
- Use concrete examples
- Vary sentence structure
- Add relevant anecdotes
- Remove redundant content

### Low Accuracy (unsupported claims)
- Add citations for facts
- Remove unsupported assertions
- Qualify uncertain statements
- Link to primary sources

## Change Feedback Format

```
✅ KEEP: rewrote introduction
   Rubric: 68→76 (+8)
   - Clarity: 15→20 (shorter sentences)
   - Engagement: 12→18 (added hook)

❌ DISCARD: expanded examples section
   Rubric: 76→72 (-4)
   - Completeness: 20→22 (+2)
   - Clarity: 20→16 (-4) ← too verbose
   - Engagement: 18→16 (-2) ← lost focus

🔄 PIVOT: engagement still low, try different approach
```

## Content Types

### Documentation
```markdown
Structure:
1. Overview (what and why)
2. Prerequisites
3. Quick Start
4. Detailed Usage
5. Examples
6. Troubleshooting
7. Reference

Tone: Clear, direct, helpful
```

### Blog Post
```markdown
Structure:
1. Hook (problem or question)
2. Context (why it matters)
3. Main content (solution or insight)
4. Examples or evidence
5. Conclusion (key takeaway)
6. Call to action

Tone: Conversational, engaging
```

### Technical Spec
```markdown
Structure:
1. Summary
2. Background/Context
3. Requirements
4. Proposed Solution
5. Alternatives Considered
6. Implementation Plan
7. Open Questions

Tone: Precise, thorough
```

## Example Session

```text
goal: Write API documentation for auth endpoints
success_criteria: rubric_score >= 85
metric_key: rubric_score
metric_direction: higher-is-better

Iteration 0: baseline outline = 35
  Completeness=10, Clarity=10, Engagement=5, Accuracy=10

Iteration 1: ✅ KEEP write endpoint descriptions
  Rubric: 35→52 (+17)

Iteration 2: ✅ KEEP add request/response examples
  Rubric: 52→68 (+16)

Iteration 3: ✅ KEEP add error codes section
  Rubric: 68→78 (+10)

Iteration 4: ❌ DISCARD verbose authentication flow
  Rubric: 78→74 (-4) ← too wordy

Iteration 5: ✅ KEEP simplified auth flow diagram
  Rubric: 78→86 (+8)

Goal met: 86 >= 85
```

## Version Control

For content, track versions explicitly:

```
outputs/
├── content-v1.md (initial)
├── content-v2.md (after iter 3)
├── content-v3.md (after iter 5)
└── content-final.md (goal met)
```

Keep previous versions until loop completes. Delete intermediates at end.

## Handoff Template

```markdown
## Content Loop Handoff

**Content Type:** [docs/blog/spec/etc.]

**Final Score:** X/100
- Completeness: X/25
- Clarity: X/25
- Engagement: X/25
- Accuracy: X/25

**Output Location:**
<session_dir>/outputs/content-final.md

**Word Count:** X words

**Sections:**
1. [section] - complete/needs work
2. [section] - complete/needs work

**Revision History:**
- v1: initial draft
- v2: improved clarity
- v3: added examples
- final: polished

**Notes:**
- [any caveats]
- [areas that could use more work]
```

## Common Pitfalls

- **Perfectionism**: Good enough is better than endless polishing
- **Losing voice**: Maintain consistent tone across revisions
- **Over-editing**: Sometimes simpler is better
- **Scope creep**: Don't expand beyond original requirements
- **Ignoring audience**: Write for the reader, not yourself
- **Format neglect**: Structure and formatting matter
