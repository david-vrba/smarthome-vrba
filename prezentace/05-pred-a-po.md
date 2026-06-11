# Srovnání PŘED a PO — SmartHome.cz

## Skóre

| Kategorie | PŘED | PO | Rozdíl |
|-----------|------|-----|--------|
| Performance (desktop) | 97 | 99–100 | +2–3 |
| **Performance (mobil)** | **71** | **99–100** | **+28–29** |
| Accessibility | ~85 | 100 | +15 |
| Best Practices | ~90 | 100 | +10 |
| SEO | ~90 | 100 | +10 |

---

## Core Web Vitals (mobil)

| Metrika | PŘED | PO | Zlepšení | Cíl |
|---------|------|-----|---------|-----|
| **LCP** | 5.9s | ~2.0s | **−66%** | < 2.5s ✅ |
| **FCP** | 3.1s | ~1.5s | **−52%** | < 1.8s ✅ |
| TBT | 0ms | 0ms | — | < 200ms ✅ |
| CLS | 0 | 0 | — | < 0.1 ✅ |
| Speed Index | 4.2s | ~2.0s | **−52%** | < 3.4s ✅ |
| TTI | 4.8s | ~2.5s | **−48%** | < 3.8s ✅ |

---

## Render-blocking zdroje

### PŘED
| Zdroj | Velikost | Ztracený čas (mobil) |
|-------|----------|---------------------|
| cdn.tailwindcss.com (JS) | 127KB | **1,524ms** |
| fonts.googleapis.com (CSS) | 1.3KB | **1,458ms** |
| **Celkem** | **128KB** | **2,982ms** |

### PO
| Zdroj | Velikost | Ztracený čas (mobil) |
|-------|----------|---------------------|
| Inline CSS (v HTML) | ~15KB (inline) | **0ms** |
| Google Fonts (async) | 1.3KB | **0ms** |
| **Celkem** | **~15KB** | **0ms** |

**Úspora: 2,982ms render-blocking čas eliminován.**

---

## Network waterfall

### PŘED (sériové blokující požadavky)
```
0ms        500ms      1000ms     1500ms     2000ms     2500ms     3000ms     3500ms     4000ms     4500ms     5000ms     5500ms     6000ms
|          |          |          |          |          |          |          |          |          |          |          |          |
├─ HTML ──┤
           ├── Tailwind CDN JS (127KB, BLOCKING) ──────────────────────┤
           ├── Google Fonts CSS (1.3KB, BLOCKING) ──────────────────┤
                                                                     ├── Inter woff2 (85KB) ───────────────────────────────────────┤
                                                                     ├── Syne woff2 (48KB) ────────────────────────────────┤
                                                                                                                          ├── LCP (5.9s) ─┤
```

### PO (paralelní, neblokující)
```
0ms        500ms      1000ms     1500ms     2000ms
|          |          |          |          |
├─ HTML + inline CSS ─┤
├── Preload hero img ──────────────────────┤
├── Google Fonts (async, NON-blocking) ────┤
├── GSAP (deferred) ───────────────────────┤
                      ├─ FCP ─┤
                                    ├─ LCP (~2.0s) ─┤
```

---

## Přístupnost

| Funkce | PŘED | PO |
|--------|------|-----|
| Skip link | ❌ Chybí | ✅ `<a href="#main-content">Přeskočit na obsah</a>` |
| Focus styly | ❌ Žádné | ✅ `focus-visible` outline |
| Reduced motion | ❌ Ignorováno | ✅ `@media (prefers-reduced-motion: reduce)` |
| ARIA labely | ❌ Chybí | ✅ `aria-label="Hlavní navigace"` |
| Sémantický HTML | ⚠️ Částečný | ✅ `<main>`, `<nav>`, `<section>`, `<footer>` |
| `<div>` v `<h1>` | ❌ Nevalidní | ✅ Nahrazeno `<span>` |
| `document.write()` | ❌ Zastaralý | ✅ Nahrazeno `<span id>` |
| Nadpisy (h1→h4 skok) | ❌ Přeskakování úrovní | ✅ `<h4>` → `<div>` (není nadpis) |

---

## Velikost přenášených dat

### PŘED
| Zdroj | Velikost |
|-------|----------|
| index.html | ~10KB |
| Tailwind CDN JS | 127KB |
| Google Fonts CSS | 1.3KB |
| Inter font | 85KB |
| Syne font | 48KB |
| GSAP | 71KB |
| Paper.js | 69KB |
| Hero image | 72KB |
| **Celkem** | **~483KB** |

### PO (desktop)
| Zdroj | Velikost |
|-------|----------|
| index.html (vč. inline CSS) | ~49KB |
| Google Fonts CSS (async) | 1.3KB |
| Inter font | 85KB |
| Syne font | 48KB |
| GSAP + ScrollTrigger | ~85KB |
| Paper.js (desktop only) | 69KB |
| Hero image | 72KB |
| **Celkem** | **~409KB** |

### PO (mobil)
| Zdroj | Velikost |
|-------|----------|
| index.html | ~49KB |
| Google Fonts CSS (async) | 1.3KB |
| Inter + Syne | 133KB |
| GSAP + ScrollTrigger | ~85KB |
| Paper.js | **0KB** (nenačítá se) |
| Hero image | **0KB** (hidden na mobilu) |
| **Celkem** | **~268KB** |

**Úspora na mobilu: 483KB → 268KB = −215KB (−45%)**

---

## Opravené bugy

| Bug | Důsledek | Oprava |
|-----|----------|--------|
| Preload URL nesedí (Unsplash vs e-spirit) | Zbytečný preload, obrázek se stahuje pozdě | Opraven URL na správný |
| Chybí ScrollTrigger plugin | Feature card animace nefungují | Přidán `ScrollTrigger.min.js` |
| `document.write()` ve footeru | Zastaralý, blokuje parser | `<span id="copyright-year">` |
| Cursor blink JS interval | Zbytečný setInterval 2x/s | CSS `@keyframes cursor-blink` |
| Transform composition (nav) | Navigace přetéká vpravo po odstranění Tailwind | `translateX(-50%) translateY(-20px)` |
| `<div>` uvnitř `<h1>` | Nevalidní HTML | `<span>` s `display: block` |
| Favicon 404 | Chybějící favicon, konzolová chyba | Inline SVG data URI |

---

## Rozšíření webu (Fáze 5) — obsah PŘED vs PO

Po optimalizaci dostal web nový obsah. Cíl: přidat sekce **bez ztráty výkonu.**

| | PŘED rozšířením | PO rozšíření |
|---|---|---|
| Řádků v `index.html` | 1020 | **1690** |
| Sekce | hero, features, footer | hero, **galerie**, features, **produkty**, footer |
| Navigace | jen desktop | **+ hamburger menu** (< 768px) |
| CTA „Konzultace" / „Poptávka" | vedlo do prázdna | **payment wall** (modal) |
| Obrázky | 1 hero (externí CDN) | hero **+ 44 lokálních variant** (galerie + produkty) |
| Formáty obrázků | JPEG | **AVIF + WebP** (`<picture>`) |
| Performance (desktop/mobil) | 99–100 / 99–100 | **98–100 / 98–100** |
| CLS | 0 | **0** (zachováno) |

### Jak zůstalo skóre nahoře i s dvojnásobkem obsahu
| Technika | Efekt |
|----------|-------|
| `<picture>` AVIF + WebP | menší obrázky než JPEG (−30 až −50 %) |
| `srcset` + `sizes` (3 velikosti) | mobil stáhne malou variantu, desktop velkou |
| `loading="lazy"` | obrázky pod foldem se nestahují předem |
| `width`/`height` + `aspect-ratio` | rezervované místo → **CLS zůstal 0** |
| Hamburger menu / payment wall JS | malé samostatné IIFE, neblokuje render |

### Responzivní obrázky — PŘED vs PO (kód)
```html
<!-- PŘED: prostý obrázek, jeden formát, jedna velikost -->
<img src="galerie1.jpg" alt="...">

<!-- PO: dva formáty, tři velikosti, lazy, pevné rozměry -->
<picture>
  <source type="image/avif" srcset="img/g1-400.avif 400w, img/g1-800.avif 800w, img/g1-1200.avif 1200w" sizes="...">
  <source type="image/webp" srcset="img/g1-400.webp 400w, img/g1-800.webp 800w, img/g1-1200.webp 1200w" sizes="...">
  <img src="img/g1-800.webp" alt="..." width="800" height="600" loading="lazy" decoding="async">
</picture>
```
