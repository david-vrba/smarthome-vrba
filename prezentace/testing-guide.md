# Testing Guide — Lighthouse for Local HTML

## Setup (already installed)
- Node.js v25.1.0
- Lighthouse v13.0.3 (`npx lighthouse`)
- `npx serve` v14.2.6

No installation needed. Run from the `SIN/` directory.

---

## Quick Test (both desktop + mobile)

```bash
cd /Users/david/Documents/skola/itk/SIN

# Start server
npx serve . -p 4321 &

# Desktop audit
npx lighthouse http://localhost:4321 \
  --output html --output json \
  --output-path ./docs/lighthouse-$(date +%Y%m%d-%H%M) \
  --chrome-flags="--headless --no-sandbox --disable-gpu" \
  --only-categories=performance \
  --preset=desktop

# Mobile audit (default throttled = simulated mid-tier 4G Android)
npx lighthouse http://localhost:4321 \
  --output html --output json \
  --output-path ./docs/lighthouse-$(date +%Y%m%d-%H%M)-mobile \
  --chrome-flags="--headless --no-sandbox --disable-gpu" \
  --only-categories=performance

# Kill server
kill $(lsof -ti:4321)

# Open report
open ./docs/lighthouse-*.report.html
```

---

## What the Presets Mean

| Flag | Throttling | Device | Use When |
|------|-----------|--------|----------|
| `--preset=desktop` | None | Desktop viewport (1350×940) | Testing desktop performance |
| (no preset) | Simulated 4G (10Mbps down, 40ms RTT) | Mobile (412×823) | Testing real-world mobile |
| `--preset=perf` | Same as mobile default | Mobile | Alias for default mobile |

**Always test both.** Desktop and mobile tell different stories.

---

## Reading the JSON Output (Quick Script)

```bash
node -e "
const fs = require('fs');
const r = JSON.parse(fs.readFileSync('./docs/FILENAME.json', 'utf8'));
const a = r.audits;
console.log('Score:', Math.round(r.categories.performance.score * 100));
['first-contentful-paint','largest-contentful-paint','total-blocking-time','cumulative-layout-shift','speed-index','interactive'].forEach(id => {
  if(a[id]) console.log(id + ':', a[id].displayValue, '(score:', Math.round((a[id].score||0)*100) + ')');
});
"
```

---

## Key Audits to Watch

| Audit ID | What it measures | Target |
|----------|-----------------|--------|
| `largest-contentful-paint` | LCP time | < 2.5s |
| `first-contentful-paint` | First paint | < 1.8s |
| `total-blocking-time` | Long JS tasks | 0ms |
| `cumulative-layout-shift` | Visual stability | 0 |
| `render-blocking-insight` | Blocking resources | 0 items |
| `unused-javascript` | Dead code | < 20KB waste |
| `lcp-breakdown-insight` | LCP waterfall | Check element render delay |

---

## Interpreting Scores

| Score | Label | CWV Status |
|-------|-------|-----------|
| 90–100 | Good (green) | Passing |
| 50–89 | Needs improvement (orange) | Failing |
| 0–49 | Poor (red) | Failing |

---

## Common Issues

**Port already in use:**
```bash
kill $(lsof -ti:4321)
```

**Chrome not found:**
```bash
# Install Chrome if needed, or use Chromium path:
npx lighthouse http://localhost:4321 --chrome-flags="--headless" --chrome-path=/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome
```

**Network requests to external CDNs fail in headless mode:**
They won't fail — Lighthouse uses real network requests. External CDNs (Tailwind, fonts, GSAP) are actually fetched during the audit, which is why the scores reflect real-world CDN latency.

---

## Naming Convention for Reports

```
docs/
  lighthouse-baseline.report.html       ← first run, before any changes
  lighthouse-baseline-mobile             ← mobile baseline (no extension = JSON only)
  lighthouse-phase1.report.html          ← after Phase 1 fixes
  lighthouse-phase1-mobile.report.html
  lighthouse-phase2.report.html
  lighthouse-final.report.html
  lighthouse-final-mobile.report.html
```
