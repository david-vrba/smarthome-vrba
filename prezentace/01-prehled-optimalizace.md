# Přehled optimalizace — SmartHome.cz

## O projektu

**SmartHome.cz** je jednostránková landing page pro fiktivní firmu zabývající se chytrými domácnostmi. Web je postaven jako **jeden soubor `index.html`** (1690 řádků, ~60KB) bez build systému, bez frameworků na straně serveru.

> **Dvě fáze projektu:** Nejdřív jsem web optimalizoval pro Core Web Vitals (Fáze 1 — skóre mobil 71 → 99). Pak jsem web **rozšířil** o nový obsah (Fáze 2 — galerie responzivních obrázků, sekce produktů s ceníkem, responzivní hamburger menu a payment wall) — a celé to musel udržet na stejném skóre. Viz sekce „Fáze 5 — Rozšíření webu" níže.

### Použité technologie
- **HTML5** — sémantická struktura
- **CSS** — inline `<style>` blok (původně Tailwind CDN, nahrazeno ručně extrahovaným CSS)
- **JavaScript** — inline `<script>` blok
- **GSAP 3.12** — animace (ScrollTrigger, timeline)
- **Paper.js** — interaktivní tekutý blob na pozadí (canvas)
- **Google Fonts** — Inter (text) + Syne (nadpisy)

### Vizuální design
- Glassmorphism (průhledné karty s blur efektem)
- Metalický gradient na textu
- Vlastní kurzor s mix-blend-mode difference
- Parallax efekt na hero sekci
- Animované feature karty (shuffler, typewriter, cursor scheduler)
- Tekutý blob na pozadí (Paper.js canvas)
- Galerie obrázků (4 responzivní `<picture>` s AVIF/WebP)
- Sekce produktů s ceníkem (5 reálných produktů, ceny v Kč)
- Responzivní hamburger menu (mobil < 768px)
- Payment wall (modální okno s formulářem)

---

## Výchozí stav (baseline)

| Metrika | Desktop | Mobil (throttled 4G) |
|---------|---------|---------------------|
| **Performance** | **97** | **71** |
| FCP | 0.8s | 3.1s |
| LCP | 1.1s | **5.9s** |
| TBT | 0ms | 0ms |
| CLS | 0 | 0 |
| Speed Index | 1.2s | 4.2s |
| Accessibility | ~85 | ~85 |
| Best Practices | ~90 | ~90 |
| SEO | ~90 | ~90 |

### Hlavní problémy
1. **Tailwind CDN blokuje vykreslování** — synchronní `<script>` stahuje 127KB JS, spustí JIT kompilaci, vygeneruje 130KB CSS → 1524ms ztraceného času na mobilu
2. **Google Fonts blokuje vykreslování** — `<link rel="stylesheet">` v `<head>` je render-blocking, řetězec: googleapis → gstatic → 1458ms ztráta
3. **Preload URL nesedí** — preload na Unsplash obrázek, ale skutečný `<img>` je z e-spirit.cloud
4. **Chybí GSAP ScrollTrigger plugin** — animace feature karet nefungují
5. **`document.write()` ve footeru** — zastaralý antipattern
6. **Chybí přístupnost** — žádný skip link, chybí focus styly, žádný prefers-reduced-motion

---

## Konečný stav (po optimalizaci)

| Metrika | Desktop | Mobil |
|---------|---------|-------|
| **Performance** | **99–100** | **99–100** |
| **Accessibility** | **100** | **100** |
| **Best Practices** | **100** | **100** |
| **SEO** | **100** | **100** |
| FCP | ~0.5s | ~1.5s |
| LCP | ~1.0s | ~2.0s |
| TBT | 0ms | 0ms |
| CLS | 0 | 0 |

### Co se zlepšilo
- **Performance mobil: 71 → 99** (+28 bodů)
- **LCP mobil: 5.9s → 2.0s** (−3.9s = −66%)
- **FCP mobil: 3.1s → 1.5s** (−1.6s = −52%)
- **Accessibility: ~85 → 100** (plná WCAG 2.1 AA shoda)
- **Best Practices: ~90 → 100**
- **SEO: ~90 → 100**
- **WAVE (přístupnost): čistý report** (0 chyb)
- **html-validate: 0 chyb**

---

## Fáze optimalizace

### Fáze 1 — Odstranění render-blokujících zdrojů (+28 bodů na mobilu)

| Oprava | Co se změnilo | Dopad |
|--------|---------------|-------|
| 1.1 Tailwind CDN → inline CSS | Nahrazeno ~190 utility tříd jako čisté CSS v `<style>` | −1500ms LCP |
| 1.2 Google Fonts → async loading | `media="print" onload="this.media='all'"` pattern | −1000ms LCP |
| 1.3 Preconnect k image hostu | `<link rel="preconnect" href="vekaprod-media...">` | −100ms LCP |
| 1.4 Oprava preload URL | Preload nyní odkazuje na správný obrázek | −200ms LCP |

### Fáze 2 — Oprava bugů a kódu

| Oprava | Co se změnilo |
|--------|---------------|
| 2.1 ScrollTrigger plugin | Přidán `ScrollTrigger.min.js` + `gsap.registerPlugin()` |
| 2.2 `document.write()` | Nahrazeno `<span id="copyright-year">` |
| 2.3 Kurzor blikání | `setInterval` → CSS `@keyframes cursor-blink` |
| 2.4 Hero text animace | `opacity-0 gsap-fade` → CSS `@keyframes fadeSlideUp` (neblokuje LCP) |

### Fáze 3 — Přístupnost (Accessibility)

| Oprava | Standard |
|--------|----------|
| Skip link (`Přeskočit na obsah`) | WCAG 2.1 AA |
| `a:focus-visible` outline styly | WCAG 2.1 AA |
| `@media (prefers-reduced-motion: reduce)` | WCAG 2.1 AA |
| `aria-label` na navigaci | WCAG 2.1 AA |
| Sémantický HTML (`<main>`, `<nav>`, `<section>`) | HTML5 |
| `<span>` místo `<div>` uvnitř `<h1>` | Validní HTML |

### Fáze 4 — Finální optimalizace

| Oprava | Dopad |
|--------|-------|
| Paper.js conditional loading (>1024px) | −67KB na mobilu |
| Oprava rozměrů obrázku (540×405 místo 1600×1067) | Rychlejší dekódování |
| `decoding="sync"` pro LCP obrázek | Prioritní dekódování |
| Favicon URL encoding | W3C validátor fix |
| CSS minifikace (odstranění komentářů) | −2KB |

---

## Fáze 5 — Rozšíření webu (nový obsah, skóre zachováno)

Po dokončení optimalizace dostal web nový obsah. Výzva: **přidat sekce, aniž by spadlo skóre.** Výsledek: stránka má víc než dvojnásobek obsahu (1020 → 1690 řádků), ale výkon zůstal na **98–100 / 98–100** a CLS pořád **0**.

### 5.1 Galerie responzivních obrázků (`#galerie`)
- 4 obrázky, každý jako `<picture>` se **dvěma moderními formáty** — AVIF (nejmenší) + WebP (fallback)
- `srcset` se 3 šířkami (400 / 800 / 1200px) + `sizes` → prohlížeč si stáhne jen tu správnou velikost
- `loading="lazy"` — obrázky pod foldem se načtou až při scrollu
- `width`/`height` + `aspect-ratio` → **nulový layout shift** (CLS zůstává 0)
- Celkem **44 obrazových variant** vygenerováno lokálně (`cwebp`, `avifenc`)

### 5.2 Sekce produktů s ceníkem (`#produkty`)
- 5 reálných produktů (identifikovaných z obrázků), reálné ceny v Kč:
  - RainPoint Wi-Fi Smart Gateway — 990 Kč
  - tado Internet Bridge — 1 190 Kč
  - tado Chytrý termostat V3+ — 2 490 Kč
  - MINIX Z100-0dB Mini PC — 6 990 Kč
  - MINIX Mini PC Wi-Fi 6 — 9 490 Kč
- Responzivní grid (1 → 2 → 3 sloupce), product obrázky opět `<picture>` AVIF/WebP
- Popisy v fontu **Inter**, tlačítko „Objednat" vede na payment wall

### 5.3 Responzivní hamburger menu
- Na desktopu klasické menu, na mobilu (< 768px) **hamburger ikona**
- Plně přístupné: `aria-expanded`, `aria-controls`, `aria-label`, animace na „X"
- Ovládá se i klávesnicí, zavírá se klávesou ESC

### 5.4 Payment wall (modální okno)
- „Konzultace" a „Nezávazná poptávka" vedou na payment wall místo do prázdna
- Přístupný modal: `role="dialog"`, `aria-modal`, **focus trap**, zavření přes ESC / klik na pozadí
- Formulář (jméno, e-mail, číslo karty s automatickým formátováním) → success obrazovka

### 5.5 Drobnost — odstranění znaku °
- Z názvů „tado°" odstraněn znak `°` (brand mark) na všech místech → jednodušší a konzistentní text

---

## Použité nástroje

| Nástroj | Účel | Výsledek |
|---------|------|----------|
| **Google Lighthouse** (v13.0.3) | Performance, Accessibility, Best Practices, SEO | 99/100/100/100 |
| **WAVE WebAIM** | Přístupnost pro postižené uživatele | Čistý report |
| **W3C Nu HTML Checker** | Validita HTML/CSS kódu | 3 falešné chyby (moderní CSS) |
| **html-validate** | Lokální HTML validátor | 0 chyb |
| **Chrome DevTools** | Debugging, Network waterfall | Analýza kritické cesty |

---

## Klíčová poučení

1. **Render-blocking zdroje jsou hlavní problém** — Tailwind CDN + Google Fonts zabíraly 3000ms na mobilu
2. **Tailwind CDN není vhodný pro produkci** — je to vývojový nástroj, ne produkční řešení
3. **CSS-only animace pro LCP elementy** — nesmí záviset na JS knihovně (GSAP `window.load` je příliš pozdě)
4. **Podmíněné načítání** — Paper.js (67KB) není potřeba na mobilu
5. **`media="print" onload` trik** — elegantní způsob asynchronního načítání fontů
6. **Přístupnost není bonus** — skip link, focus styly a prefers-reduced-motion jsou standard
7. **Přidat obsah ≠ zhoršit výkon** — galerie i produkty mají moderní formáty (AVIF/WebP), `loading="lazy"` a pevné rozměry → web zdvojnásobil obsah a skóre zůstalo 98–100, CLS 0
8. **Responzivní obrázky se vyplatí** — `<picture>` + `srcset`/`sizes` ušetří mobilním uživatelům stahování velkých obrázků
