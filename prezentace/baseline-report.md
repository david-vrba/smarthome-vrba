# Baseline Lighthouse Report — SmartHome.cz

**Date:** 2026-03-25
**Lighthouse version:** 13.0.3
**URL tested:** http://localhost:4321 (served via `npx serve`)
**File:** index.html (817 lines, ~10KB HTML + inline CSS/JS)

---

## Scores Summary

| Category | Desktop | Mobile (Throttled 4G) |
|----------|---------|----------------------|
| **Performance** | **97** | **71** |

## Core Web Vitals

| Metric | Desktop Score | Desktop Value | Mobile Score | Mobile Value | Target |
|--------|--------------|---------------|-------------|--------------|--------|
| FCP (First Contentful Paint) | 96 | 0.8s | 45 | 3.1s | < 1.8s |
| LCP (Largest Contentful Paint) | 93 | 1.1s | **14** | **5.9s** | < 2.5s |
| TBT (Total Blocking Time) | 100 | 0ms | 100 | 0ms | < 200ms |
| CLS (Cumulative Layout Shift) | 100 | 0 | 100 | 0 | < 0.1 |
| Speed Index | 93 | 1.2s | 78 | 4.2s | < 3.4s |
| TTI (Time to Interactive) | 100 | 1.1s | 79 | 4.8s | < 3.8s |

## Failed Audits

### Desktop
| Audit | Score | Details |
|-------|-------|---------|
| Render-blocking resources | 0 | Tailwind CDN: 369ms wasted; Google Fonts: 315ms wasted |
| Unused JavaScript | 50 | Paper.js: 57KB wasted; Tailwind CDN: 38KB wasted (94KB total) |
| Cache policy | 0 | External CDN resources lack cache control (expected for CDNs) |
| Image delivery | 0 | Hero image: 72KB, could save 44KB with WebP/AVIF |

### Mobile (Throttled)
| Audit | Score | Details |
|-------|-------|---------|
| Render-blocking resources | 0 | **Tailwind CDN: 1,524ms wasted; Google Fonts CSS: 1,458ms wasted** |
| LCP breakdown | 0 | TTFB: 4ms — Element Render Delay: **5,939ms** |
| Unused JavaScript | 0 | Est. 56KB savings |
| Cache policy | 0 | As above |
| Network dependency chain | 0 | Font chain: localhost → fonts.googleapis.com → fonts.gstatic.com (serial) |

## Root Cause Analysis

### Why is mobile LCP 5.9s?

The LCP element on mobile is **`<p class="text-lg">`** (the hero paragraph text), not the image (which is `hidden lg:block` on mobile). The render delay is **5.939 seconds** caused by:

```
Network Chain (critical path):
localhost (10ms)
  └── fonts.googleapis.com CSS (1,451ms total nav time)
        └── fonts.gstatic.com Inter woff2 (4,701ms total nav time) ← LONGEST
        └── fonts.gstatic.com Syne woff2 (4,503ms)
  └── cdn.tailwindcss.com (1,524ms blocking)
```

**Tailwind CDN blocks render** because it's a synchronous `<script>` that:
1. Downloads ~127KB of JS
2. Executes JIT compilation against the entire DOM
3. Injects ~130KB of generated CSS
4. Only then does the browser paint

**Google Fonts blocks render** because the `<link rel="stylesheet">` in `<head>` is render-blocking by default, and the font chain requires two serial network round trips (Google API → gstatic CDN).

### Why is CLS 0 despite all the `opacity-0` + GSAP animations?
Opacity and transform changes don't contribute to CLS (by design — the browser counts layout-affecting shifts only). The `translate-y-8` on feature cards is out of viewport during initial load so it doesn't register.

### Why is TBT 0ms?
The JS execution time is minimal — GSAP and Paper.js are both deferred, so they don't block the main thread during page load. All long tasks happen post-load.

## Third-Party Resources Loaded

| Resource | Size | Blocking | Purpose |
|----------|------|----------|---------|
| cdn.tailwindcss.com | 127KB | **Yes** | CSS framework (JIT) |
| fonts.googleapis.com | 1.3KB | **Yes** | Font CSS |
| fonts.gstatic.com (Inter) | 85KB | No (font) | Inter font |
| fonts.gstatic.com (Syne) | ~48KB | No (font) | Syne font |
| cdnjs.cloudflare.com/gsap | ~71KB | No (defer) | Animations |
| cdnjs.cloudflare.com/paper.js | 69KB | No (defer) | Liquid canvas |
| vekaprod-media.e-spirit.cloud | 72KB | No (eager img) | Hero image |

**Total transfer size:** ~481KB (desktop, no throttling)

## Additional Bugs Found (Not CWV but Should Fix)

1. **Preload URL mismatch (line 19):** `<link rel="preload" as="image" href="https://images.unsplash.com/...">` — the preloaded URL is a different image than the actual hero `<img src>` (e-spirit.cloud). Preload is wasted.

2. **Missing GSAP ScrollTrigger plugin (line 628-638):** `scrollTrigger: { trigger: '#features' }` is used inside `gsap.to()` but only GSAP core is loaded. ScrollTrigger plugin is never imported. Feature card scroll animations are silently broken.

3. **`document.write()` in footer (line 539):** `<script>document.write(new Date().getFullYear())</script>` — parser-blocking, deprecated antipattern.

4. **Typewriter setInterval at 50ms (lines 686-693):** 20 DOM mutations per second on main thread.

5. **Cursor blink via setInterval (lines 695-698):** Another interval, better handled with CSS animation.
