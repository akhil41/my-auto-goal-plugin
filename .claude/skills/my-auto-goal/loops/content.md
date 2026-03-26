# Content Domain Loop

Guidance for `domain_type: content` loops.

## Scope

Write and revise content. No code changes, no technical implementation.

## Rubric (Self-Scoring)

| Dimension | 0-5 (Absent) | 6-12 (Weak) | 13-18 (Adequate) | 19-22 (Strong) | 23-25 (Excellent) |
|-----------|-------------|-------------|-------------------|-----------------|-------------------|
| Completeness | No content or major sections missing | Most topics touched but thin | All required topics covered, some gaps | Thorough coverage with examples | Comprehensive — nothing missing, good depth |
| Accuracy | Unsupported claims, factual errors | Some verified facts, many unqualified | Mostly accurate, key claims supported | Well-researched, sources noted | All claims verified, caveats where uncertain |
| Clarity | Disorganized, jargon-heavy | Structure exists but hard to follow | Clear structure, some rough spots | Easy to read, good flow, terms defined | Effortless to read, strong transitions |
| Actionability | No next steps, abstract only | Some advice but vague | Concrete recommendations for some personas | Clear next steps for most readers | Copy-paste examples, decision tools, all personas covered |

**Scoring rules:**
- Score each dimension independently using the bands above
- Compare directly to the previous version — did THIS dimension improve?
- Total = sum of all four dimensions (0-100)

## Revision Strategy

Target the **weakest dimension** each iteration:

| Low dimension | Fix |
|--------------|-----|
| Completeness | Add missing sections, expand thin areas, add examples |
| Accuracy | Verify claims with WebSearch/WebFetch, add source notes, remove unsupported claims, qualify uncertain statements |
| Clarity | Shorter sentences (≤20 words), active voice, transitions, define jargon on first use |
| Actionability | Add code examples, decision flowcharts, concrete next steps, persona-specific recommendations, copy-paste commands |

## Factual Verification

If the content makes factual claims (prices, benchmarks, statistics), verify them:
1. Use WebSearch to find current data
2. Use WebFetch on official documentation pages
3. If web tools fail, see `loops/research.md` for fallback strategies
4. Mark confidence level: "verified from [source]" or "approximate — verify before decisions"

## Iteration Pattern

1. Self-score current state (record all 4 dimension scores)
2. Identify weakest dimension
3. Make ONE focused revision targeting it
4. Re-score → keep if improved, discard if not
5. Log to results.tsv AND record dimension breakdown in session.md:
```
✅ KEEP: [revision] — Rubric: X→Y (+Z)
   Completeness: 20→22, Accuracy: 16→21, Clarity: 21→22, Actionability: 20→21
❌ DISCARD: [revision] — Rubric: X→Y (-Z)
   Accuracy: 21→19 (introduced unverified claims)
```

## Version Control

Track versions in `<session_dir>/outputs/`: `content-v1.md`, `content-v2.md`, ..., `content-final.md`.

- `<session_dir>` = the `.autogoal-<YYYYMMDD-HHMMSS>/` directory created in Phase 3
- Keep all previous versions until loop completes
- **On success:** Copy the best-scoring version to `content-final.md` in the same `outputs/` directory

## Baseline (Iteration 0)

If no content exists yet, baseline score is 0 across all dimensions. Log:
```
iter=0, score=0, status=baseline, description="No content yet"
```
Do NOT try to evaluate empty content against the rubric.
