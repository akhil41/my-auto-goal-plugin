# Web-SEO Domain Loop

Guidance for `domain_type: web-seo` loops.

## Scope

Audit and improve web performance, SEO, and accessibility metrics. No major feature changes, no unrelated refactoring.

## Quality Gate

| Check | Pass criteria |
|-------|--------------|
| Build passes | Exit 0 |
| Page loads | 200 OK |
| No regressions | No category dropped > 5 points |

## Audit-First Pattern

Always audit before changing:
1. Run Lighthouse → parse scores
2. Identify LOWEST scoring audit item
3. Implement ONE fix
4. Re-audit → keep if improved

## Priority Order (fix highest impact first)

```
🔴 Critical: render-blocking resources, missing viewport, no HTTPS, broken links
🟡 High:     images without dimensions (CLS), large images (LCP), missing alt text, missing meta description
💭 Medium:   unused CSS/JS, heading hierarchy, color contrast, missing structured data
✨ Polish:   font optimization, preconnect hints, service worker
```

## Lighthouse Integration

```bash
npx lighthouse http://localhost:3000 --output=json --output-path=./lighthouse.json --chrome-flags="--headless"

# Extract scores
cat lighthouse.json | jq '{performance: .categories.performance.score*100, seo: .categories.seo.score*100, accessibility: .categories.accessibility.score*100}'
```

## Iteration Pattern

1. Run Lighthouse audit
2. Pick ONE fix for worst audit item
3. Implement → re-audit
4. Decision: keep if score improved, discard if worse
5. Log with full score breakdown:
```
✅ KEEP: add image dimensions
   Performance: 72→78 (+6), CLS: 0.25→0.08
❌ DISCARD: lazy load hero
   Performance: 78→71 (-7), LCP: 2.1s→3.4s
```
