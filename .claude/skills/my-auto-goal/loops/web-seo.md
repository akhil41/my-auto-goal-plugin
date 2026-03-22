# Web-SEO Domain Loop

Domain-specific guidance for `domain_type: web-seo` loops.

## Scope

Audit and improve web performance, SEO, and accessibility metrics.

**Allowed:**
- Modify HTML structure for SEO
- Optimize images and assets
- Improve meta tags and structured data
- Fix accessibility issues
- Optimize Core Web Vitals
- Improve page load performance

**Not allowed:**
- Major feature changes
- Backend logic changes (unless for performance)
- Content strategy changes (that's `content` domain)
- Unrelated code refactoring

## Quality Gate

Before keeping any iteration:

| Check | Command | Pass Criteria |
|-------|---------|---------------|
| Build passes | `npm run build` | Exit code 0 |
| No console errors | Check dev tools | Clean |
| Page loads | `curl -I URL` | 200 OK |
| No regressions | Compare scores | No category dropped >5 |

**Gate failure handling:**
- Score dropped → DISCARD immediately
- Build failed → Retry fix (max 3)
- Escalate if 3 consecutive hypothesis failures

## Typical Metrics

| Metric | Direction | Example eval_command |
|--------|-----------|---------------------|
| Lighthouse Performance | higher-is-better | `npx lighthouse URL --output=json` |
| Lighthouse SEO | higher-is-better | `npx lighthouse URL --only-categories=seo` |
| Lighthouse Accessibility | higher-is-better | `npx lighthouse URL --only-categories=accessibility` |
| LCP | lower-is-better | Extract from Lighthouse JSON |
| CLS | lower-is-better | Extract from Lighthouse JSON |
| FID/INP | lower-is-better | Extract from Lighthouse JSON |
| Page size | lower-is-better | `curl -sI URL \| grep content-length` |

## Audit-First Pattern

For web-seo, always audit before changing:

```
1. Run Lighthouse audit
2. Identify LOWEST scoring audit item
3. Research the specific fix
4. Implement ONE fix
5. Re-audit
6. Keep if improved, discard if not
```

## Iteration Pattern

```
1. Run Lighthouse audit (baseline or current)
2. Parse scores and identify worst audit
3. Pick ONE specific fix for that audit
4. Implement the change
5. Re-run Lighthouse
6. Decision:
   - Score improved → git add + commit (KEEP)
   - Score unchanged but simpler → git add + commit (KEEP)
   - Score unchanged/worse and not simpler → git restore . && git clean -fd (DISCARD)
   - Build broken → retry (max 3)
7. Log to results.tsv with all category scores
8. Update resume.md
9. Continue until goal met
```

## Priority Order

Fix issues in this order (highest impact first):

```
🔴 Critical (fix first):
- Render-blocking resources
- Missing meta viewport
- No HTTPS
- Broken links (404s)

🟡 High (fix second):
- Images without dimensions (CLS)
- Large images (LCP)
- Missing alt text (a11y)
- Missing meta description (SEO)

💭 Medium (fix third):
- Unused CSS/JS
- Missing heading hierarchy
- Color contrast issues
- Missing structured data

✨ Polish (fix last):
- Font optimization
- Preconnect hints
- Service worker
- HTTP/2 push
```

## Lighthouse Integration

```bash
# Run full audit
npx lighthouse http://localhost:3000 \
  --output=json \
  --output-path=./lighthouse.json \
  --chrome-flags="--headless"

# Extract scores (use jq)
cat lighthouse.json | jq '{
  performance: .categories.performance.score * 100,
  accessibility: .categories.accessibility.score * 100,
  seo: .categories.seo.score * 100,
  bestPractices: .categories["best-practices"].score * 100
}'

# Extract Core Web Vitals
cat lighthouse.json | jq '{
  lcp: .audits["largest-contentful-paint"].numericValue,
  cls: .audits["cumulative-layout-shift"].numericValue,
  fid: .audits["max-potential-fid"].numericValue
}'
```

## Change Feedback Format

Log with full score breakdown:

```
✅ KEEP: add image dimensions
   Performance: 72→78 (+6)
   Accessibility: 89→89 (=)
   SEO: 91→91 (=)
   CLS: 0.25→0.08 (-0.17)

❌ DISCARD: lazy load hero image
   Performance: 78→71 (-7)  ← regression
   LCP: 2.1s→3.4s (+1.3s)  ← worse

🔄 RETRY (2/3): preconnect to CDN
   Build failed: invalid link tag
```

## Example Session

```text
goal: Achieve Lighthouse SEO score >= 95
eval_command: npx lighthouse http://localhost:3000 --only-categories=seo --output=json | jq '.categories.seo.score * 100'
metric_key: seo_score
metric_direction: higher-is-better
success_criteria: seo_score >= 95

Iteration 0: baseline
  Performance=65, SEO=72, A11y=81

Iteration 1: ✅ KEEP add meta description to all pages
  SEO: 72→78 (+6)

Iteration 2: ✅ KEEP add alt text to all images
  SEO: 78→84 (+6), A11y: 81→88 (+7)

Iteration 3: ✅ KEEP fix heading hierarchy
  SEO: 84→89 (+5)

Iteration 4: ✅ KEEP add canonical URLs
  SEO: 89→92 (+3)

Iteration 5: ✅ KEEP add JSON-LD structured data
  SEO: 92→96 (+4)

Goal met: 96 >= 95
```

## Quick Fixes by Category

### Performance (LCP, Speed)
```html
<!-- Preload critical assets -->
<link rel="preload" href="hero.webp" as="image">

<!-- Defer non-critical JS -->
<script defer src="analytics.js"></script>

<!-- Lazy load below-fold images -->
<img loading="lazy" src="footer.webp">
```

### SEO
```html
<!-- Unique per page -->
<title>Page Title | Site Name</title>
<meta name="description" content="Unique 150-160 char description">
<link rel="canonical" href="https://example.com/page">

<!-- Structured data -->
<script type="application/ld+json">
{"@context":"https://schema.org","@type":"Article",...}
</script>
```

### Accessibility
```html
<!-- Images -->
<img alt="Descriptive text" src="photo.jpg">

<!-- Forms -->
<label for="email">Email</label>
<input id="email" type="email">

<!-- Navigation -->
<nav aria-label="Main navigation">
<main role="main">
```

### CLS (Layout Shift)
```html
<!-- Always set dimensions -->
<img width="800" height="600" src="photo.jpg">

<!-- Reserve space for dynamic content -->
<div style="min-height: 300px">
  <!-- dynamic content here -->
</div>
```

## Handoff Template

```markdown
## Web-SEO Loop Handoff

**Final Scores:**
| Category | Baseline | Final | Delta |
|----------|----------|-------|-------|
| Performance | X | Y | +Z |
| SEO | X | Y | +Z |
| Accessibility | X | Y | +Z |

**Core Web Vitals:**
| Metric | Baseline | Final | Status |
|--------|----------|-------|--------|
| LCP | Xs | Ys | ✅/❌ |
| CLS | X | Y | ✅/❌ |
| INP | Xms | Yms | ✅/❌ |

**Changes Made:**
1. [change] — [impact]
2. [change] — [impact]

**Remaining Opportunities:**
- [audit item still failing]

**Notes:**
- [any caveats about measurements]
```

## Common Pitfalls

- **Testing on dev vs prod**: Scores differ between environments
- **Ignoring mobile**: Always test mobile viewport
- **Single measurement**: Run 3+ audits and average
- **Caching issues**: Clear cache between measurements
- **Regression blindness**: Always compare ALL categories, not just target
