# CWV Improvement Plan — SmartHome.cz

**Goal:** Lighthouse Performance ≥ 90 on mobile, maintain ≥ 97 on desktop
**Starting point:** Desktop 97, Mobile 71
**Created:** 2026-03-25

---

## Target Scores

| Metric | Current Desktop | Current Mobile | Target Desktop | Target Mobile |
|--------|----------------|----------------|----------------|---------------|
| Performance | 97 | 71 | ≥ 97 | **≥ 90** |
| FCP | 0.8s | 3.1s | ≤ 0.8s | **≤ 1.8s** |
| LCP | 1.1s | 5.9s | ≤ 1.5s | **≤ 2.5s** |
| TBT | 0ms | 0ms | 0ms | 0ms |
| CLS | 0 | 0 | 0 | 0 |

---

## Phase 1 — Eliminate Render-Blocking Resources (Expected: +15–20 mobile points)

These two fixes alone account for **~3,000ms of wasted render-blocking time** on mobile. They are the single biggest lever.

### Fix 1.1 — Tailwind CDN → Non-blocking
**Problem:** `<script src="https://cdn.tailwindcss.com">` blocks rendering for 1,524ms on mobile.
**Root cause:** Synchronous script tag in `<head>` must execute before browser can paint.

**Solution:** Replace with a pre-extracted inline `<style>` block containing only the Tailwind utility classes actually used by the page. No CDN, no runtime JIT.

**How:**
1. Run `npx tailwindcss -i ./tailwind-input.css -o ./tailwind-out.css --content ./index.html --minify` (or do it manually by scanning class usage)
2. Inline the resulting CSS into a `<style>` tag
3. Remove the `<script src="cdn.tailwindcss.com">` and the inline `tailwind.config` script block

**Expected saving:** ~1,500ms on mobile LCP, ~370ms on desktop
**Risk:** Must capture all used classes, including dynamic ones added by JS (shuffler cards, cursor scheduler)

**Tailwind classes used in JS (must be included):**
- `bg-white/70 backdrop-blur-md border border-metal-200 p-6 rounded-[2rem] shadow-sm transition-all duration-700 ease-[cubic-bezier(0.34,1.56,0.64,1)]` (shuffler cards, line 660)
- Standard utility classes from static HTML (scan full file)

---

### Fix 1.2 — Google Fonts → Non-render-blocking
**Problem:** `<link rel="stylesheet" href="fonts.googleapis.com/...">` blocks rendering for 1,458ms on mobile. The font chain (googleapis → gstatic) requires serial round trips.

**Solution A (recommended):** Download and inline the font CSS, use `@font-face` with `font-display: swap` pointing to Google's CDN directly. This eliminates the render-blocking stylesheet while keeping CDN delivery.

**Solution B:** Add `media="print" onload="this.media='all'"` trick to load fonts asynchronously, with system font fallback.

**Solution C:** Self-host the fonts (download woff2 files, serve locally). Only viable if the file is ever served from a real server.

**Implementation (Solution A — font-display inline):**
```html
<!-- Remove this: -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Syne:wght@600;700;800&display=swap" rel="stylesheet">

<!-- Add this instead (non-blocking load + noscript fallback): -->
<link rel="preload" href="https://fonts.gstatic.com/s/inter/v20/UcC73FwrK3iLTeHuS_nVMrMxCp50SjIa25L7W0Q5n-wU.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="https://fonts.gstatic.com/s/syne/v23/8vIU7ww63nmS_OQtWwF4FQ.woff2" as="font" type="font/woff2" crossorigin>
<!-- Async load font CSS -->
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Syne:wght@600;700;800&display=swap" media="print" onload="this.media='all'">
<noscript><link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Syne:wght@600;700;800&display=swap"></noscript>
```

**Expected saving:** ~1,000–1,500ms on mobile LCP
**Risk:** Brief FOUT (Flash of Unstyled Text) with system fonts until web fonts load. Acceptable.

---

### Fix 1.3 — Add preconnect to hero image host
**Problem:** No `preconnect` to `vekaprod-media.e-spirit.cloud` (the actual hero image server).
**Fix:** Add `<link rel="preconnect" href="https://vekaprod-media.e-spirit.cloud">` in `<head>`.
**Expected saving:** ~100–200ms on LCP (eliminates DNS + TCP setup time for image)

---

### Fix 1.4 — Fix preload URL mismatch
**Problem:** Line 19 preloads `https://images.unsplash.com/...` but hero img src is `vekaprod-media.e-spirit.cloud/...`. Wasted preload.
**Fix:** Change preload href to match actual hero image URL.
**Expected saving:** ~200–400ms on LCP (browser can start fetching image earlier in waterfall)

---

## Phase 2 — Fix Broken Features & Code Bugs (No CWV impact, but important)

### Fix 2.1 — Add GSAP ScrollTrigger plugin
**Problem:** `gsap.to('.feature-card', { scrollTrigger: {...} })` on line 628 uses ScrollTrigger but the plugin is never loaded. Feature card scroll animations are silently broken.
**Fix:** Add `<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js" defer></script>` and `gsap.registerPlugin(ScrollTrigger)` before usage.
**Risk:** Low — only adds the plugin that was already expected

### Fix 2.2 — Replace `document.write()` with DOM injection
**Problem:** `<script>document.write(new Date().getFullYear())</script>` in footer (line 539).
**Fix:** `<span id="year"></span>` + `document.getElementById('year').textContent = new Date().getFullYear()`

### Fix 2.3 — Replace typewriter setInterval with requestAnimationFrame
**Problem:** `setInterval(() => { ... }, 50)` fires 20 times/sec, each time mutating DOM. Any user interaction during this can cause INP spikes.
**Fix:** Use `requestAnimationFrame` loop that self-schedules and checks a timestamp for 50ms pacing. Or better: convert to pure CSS `@keyframes` typing animation.

### Fix 2.4 — Replace cursor blink setInterval with CSS animation
**Problem:** `setInterval(() => { typeCursor.classList.toggle(...) }, 500)` — trivial fix.
**Fix:** CSS `@keyframes blink { 50% { opacity: 0; } }` applied to `#typewriter-cursor`.

---

## Phase 3 — Image Optimization (Expected: +2–3 mobile points)

### Fix 3.1 — Hero image format
**Problem:** Hero image is a JPEG at 72KB. Lighthouse flags it for WebP/AVIF conversion (44KB potential savings).
**Constraint:** External image URL — can't convert. However, if a local copy or different CDN URL is used, this could be addressed.
**Current status:** Skip unless image is hosted locally.

### Fix 3.2 — Font subsetting
**Problem:** Loading 6 font weights (Inter: 300/400/500/600, Syne: 600/700/800). Actual usage:
- Inter: 300 (light body), 400 (body), 500 (medium nav), 600 (semibold labels)
- Syne: 600, 700, 800 (headings)
Most weights are genuinely used. Minimal saving available.
**Action:** Verify which weights are actually rendered, remove unused ones from the Google Fonts URL.

---

## Phase 4 — Performance Polish (Expected: +1–2 mobile points)

### Fix 4.1 — Lazy load Paper.js on mobile
**Problem:** Paper.js (~69KB) is loaded on all devices but the liquid canvas is only meaningfully visible on large screens (the canvas exists but the animation is subtle/background on mobile).
**Fix:** Conditionally load Paper.js only on `window.innerWidth > 768` via dynamic import or `if` guard.

### Fix 4.2 — dns-prefetch for CDN domains
**Fix:** Add `<link rel="dns-prefetch" href="https://cdnjs.cloudflare.com">` for GSAP/Paper.js CDN.

### Fix 4.3 — Throttle Paper.js to 30fps
**Problem:** `view.onFrame` runs at 60fps doing 20 trig calculations + path smoothing.
**Fix:** Add frame counter, skip odd frames: `if (++frameCount % 2 !== 0) return;`

---

## Implementation Order

```
Phase 1.4 → 1.3 → 1.2 → 1.1 → Test → Phase 2 → Test → Phase 3 → Phase 4 → Final test
```

Run Lighthouse after each phase to measure actual improvement.

---

## Expected Final Scores (Estimated)

| Fix | Mobile LCP | Mobile Score |
|-----|-----------|-------------|
| Baseline | 5.9s | 71 |
| After Phase 1 | ~2.0–2.5s | ~87–90 |
| After Phase 2 | ~2.0–2.5s | ~87–90 (same CWV, bugs fixed) |
| After Phase 3+4 | ~1.8–2.2s | ~90–93 |

---

## Test After Each Phase

```bash
npx serve /Users/david/Documents/skola/itk/SIN -p 4321 &
npx lighthouse http://localhost:4321 --output html --output-path ./docs/lighthouse-phaseN --chrome-flags="--headless --no-sandbox" --only-categories=performance
npx lighthouse http://localhost:4321 --output html --output-path ./docs/lighthouse-phaseN-mobile --chrome-flags="--headless --no-sandbox" --only-categories=performance --preset=perf
kill $(lsof -ti:4321)
```
