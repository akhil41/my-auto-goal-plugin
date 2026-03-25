# Content Domain Loop

Guidance for `domain_type: content` loops.

## Scope

Write and revise content. No code changes, no technical implementation.

## Rubric (Self-Scoring)

| Dimension | 0-25 | Criteria |
|-----------|------|----------|
| Completeness | 0-25 | Covers all required topics? |
| Clarity | 0-25 | Easy to understand? |
| Engagement | 0-25 | Interesting to read? |
| Accuracy | 0-25 | Facts correct, claims supported? |

Score consistently. Compare directly to previous version.

## Revision Strategy

Target the **weakest dimension** each iteration:

| Low dimension | Fix |
|--------------|-----|
| Completeness | Add missing sections, expand thin areas, add examples |
| Clarity | Shorter sentences (≤20 words), active voice, transitions, define jargon |
| Engagement | Compelling hook, concrete examples, vary structure, cut redundancy |
| Accuracy | Add citations, remove unsupported claims, qualify uncertain statements |

## Iteration Pattern

1. Self-score current state
2. Identify weakest dimension
3. Make ONE focused revision targeting it
4. Re-score → keep if improved, discard if not
5. Log:
```
✅ KEEP: [revision] — Rubric: X→Y (+Z)
   Clarity: 15→20, Engagement: 12→18
❌ DISCARD: [revision] — Rubric: X→Y (-Z)
```

## Version Control

Track versions in `outputs/`: `content-v1.md`, `content-v2.md`, ..., `content-final.md`. Keep previous until loop completes.
