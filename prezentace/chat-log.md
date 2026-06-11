# Chat Log — SmartHome.cz Prezentace Session

**Datum:** 2026-03-26
**Kontext:** Pokračování z předchozí konverzace (optimalizace CWV), tato session se zaměřila na dokumentaci a prezentační web.

---

## Co se stalo v předchozí session (summary)

### Hlavní úkol
Optimalizace SmartHome.cz (single-file HTML landing page) pro Core Web Vitals. Cíl: Lighthouse Performance ≥ 90 na mobilu.

### Výsledky optimalizace
- **Performance mobil: 71 → 99–100** (+28 bodů)
- **LCP mobil: 5.9s → 2.0s** (−66%)
- **Accessibility: ~85 → 100**
- **Best Practices: ~90 → 100**
- **SEO: ~90 → 100**
- **WAVE: 0 chyb**
- **html-validate: 0 chyb**
- **W3C: 3 false positives (validní moderní CSS)**

### Provedené opravy (Phase 1–4)
1. **Tailwind CDN → inline CSS** — 127KB synchronní script nahrazen ~190 ručně extrahovanými utility třídami (~15KB inline CSS). Úspora: −1524ms LCP.
2. **Google Fonts → async** — `media="print" onload="this.media='all'"` pattern. Úspora: −1458ms LCP.
3. **Preload URL fix** — preload odkazoval na Unsplash, opraveno na e-spirit.cloud (skutečný hero obrázek).
4. **Preconnect** — přidán pro image host + dns-prefetch pro CDN.
5. **ScrollTrigger plugin** — chyběl, feature card animace nefungovaly.
6. **Hero text CSS animace** — `opacity-0 gsap-fade` → CSS `@keyframes fadeSlideUp` (neblokuje LCP).
7. **document.write()** → `<span id="copyright-year">2026</span>`.
8. **Cursor blink** — setInterval → CSS `@keyframes cursor-blink`.
9. **Paper.js conditional** — načítá se jen na desktop >1024px (−67KB na mobilu).
10. **Image dimensions** — 1600×1067 → 540×405 (skutečné rozměry), `decoding="sync"`.
11. **Favicon** — inline SVG data URI, URL-encoded.
12. **CSS minifikace** — odstranění komentářů (−2KB).
13. **Přístupnost** — skip link, focus-visible, prefers-reduced-motion, aria-label, sémantický HTML, `<span>` místo `<div>` v `<h1>`.

### Opravené bugy
- Navbar overflow vpravo (transform composition — `translateX(-50%) translateY(-20px)`)
- Hero text ztratil animaci (CSS @keyframes místo GSAP dependency)
- Mobile LCP regrese na 4.4s (opacity-0 čekalo na GSAP window.load)
- paper.install(window) ReferenceError (voláno před načtením Paper.js)
- Duplicate class attribute na SVG
- `<div>` uvnitř `<h1>` (HTML validace)
- Heading hierarchy skip (h4 → div)

---

## Tato session — Dokumentace & Prezentační web

### 1. Vytvoření dokumentace (docs/prezentace/)

Vytvořeno 6 souborů:

#### 01-prehled-optimalizace.md (136 řádků)
- Kompletní přehled projektu
- Výchozí stav vs. konečný stav (tabulky)
- Všechny 4 fáze optimalizace
- Použité nástroje
- Klíčová poučení

#### 02-technicka-dokumentace.md (442 řádků)
- Struktura souboru index.html (ASCII strom)
- **Pořadí načítání po milisekundách** (waterfall od 0ms do 2000ms)
- 10 klíčových kódových částí detailně vysvětlených:
  1. Tailwind CDN nahrazení
  2. Asynchronní načítání fontů (media="print" trik)
  3. CSS-only animace pro LCP element
  4. Transform composition bug
  5. Podmíněné načítání Paper.js
  6. Content-visibility optimalizace
  7. Glassmorphism efekt
  8. Liquid Metal tlačítko (@property + conic-gradient)
  9. GSAP Timeline
  10. Přístupnost (skip link, focus-visible, reduced-motion)

#### 03-nastroje-a-vysledky.md (193 řádků)
- Lighthouse — co to je, jak jsme ho použili, výsledky, váhy metrik
- WAVE WebAIM — co testuje, náš výsledek
- W3C Nu HTML Checker — 1 opravená + 3 false positive chyby
- html-validate — 0 chyb
- Chrome DevTools — Network waterfall analýza
- Core Web Vitals vysvětlení pro prezentaci (LCP, CLS, INP)

#### 04-body-prezentace.md (180 řádků)
- 10-sekční struktura prezentace s časy
- Rozdělení práce (David vs. kamarád)
- Taby k přípravě v Chrome (7 tabů)
- Body k zmínění v každé sekci
- Tipy (spustit server předem, vysvětlit variabilitu skóre)

#### 05-pred-a-po.md (143 řádků)
- Srovnávací tabulky (skóre, CWV metriky)
- Render-blocking zdroje před/po
- Network waterfall ASCII diagramy (blocking vs. parallel)
- Přístupnost před/po
- Velikost přenášených dat (483KB → 268KB na mobilu = −45%)
- Opravené bugy

#### index-komentovany.html (1557 řádků)
- Kompletní kopie index.html s českými komentáři
- Každá sekce vysvětlena: co to je, jak to funguje, proč je to tam
- Komentáře pokrývají:
  - Meta tagy a jejich účel
  - Preload/preconnect mechanismy
  - Async font loading trik
  - defer vs async vs blocking
  - Tailwind utility třídy (organizace, jak fungují)
  - CSS custom properties a transform systém
  - Glassmorphism (backdrop-filter, rgba, inset shadow)
  - @property a conic-gradient (liquid metal tlačítko)
  - Content-visibility + contain-intrinsic-size
  - CSS vs GSAP animace pro LCP
  - Skip link, focus-visible, prefers-reduced-motion
  - GSAP timeline (offsety, stagger, yoyo)
  - Paper.js (podmíněné načítání, blob generování, lerp)
  - Shuffler, Typewriter, Cursor Scheduler logika

### 2. Prezentační web (docs/prezentace/prezentace.html)

**1335 řádků, čistý HTML/CSS/JS, žádné závislosti.**

13 scroll-snap sekcí:

1. **Title** — gradient nadpis, jména, scroll indikátor
2. **Client & Business Identity** — client brief karta, proč simulovat klienta, tech stack badges
3. **Co je SmartHome.cz** — 4 feature karty (glassmorphism, GSAP, Paper.js, kurzor)
4. **Proč CWV** — 3 metriky (LCP/CLS/TBT) s prahy, statistiky (53% odchod, −7% konverze)
5. **Před — Problém** — animovaný counter (71), červený waterfall diagram s blocking řetězcem
6. **Co jsme opravili** — 4 fix karty s impact badges
7. **Kód před vs po** — tabbed code comparison (Tailwind, Fonts, Hero) se syntax highlightingem
8. **Po — Výsledky** — animovaný counter (99), zelený waterfall, +28 bodů
9. **Scorecard** — 4 SVG progress ringy (99/100/100/100) s animacemi
10. **Nástroje** — 5 tool karet (Lighthouse, WAVE, W3C, html-validate, DevTools)
11. **Přístupnost** — checklist grid s fajfkami
12. **Klíčová poučení** — 5 numbered takeaway karet
13. **Q&A** — závěrečný slide

Interaktivní funkce:
- Scroll-snap (každá sekce = jedna obrazovka)
- Navigační tečky (pravá strana, klikatelné)
- Animované počítadla (count up při scrollu)
- Animované waterfall bary (rostou při scrollu)
- SVG progress ring animace
- Tab přepínání pro code comparison
- IntersectionObserver-triggered reveal animace
- prefers-reduced-motion respektováno

Design:
- Dark theme (#0f172a slate)
- Accent: emerald (#10b981) pro "good", red (#ef4444) pro "bad"
- Monospace pro kód, system font stack pro text
- Karty s hover efektem (border glow)

---

## Struktura souborů (aktuální)

```
SIN/
├── index.html                              ← hlavní web (1020 řádků, 49KB)
├── CLAUDE.md                               ← project instructions
├── docs/
│   ├── baseline-report.md                  ← výchozí audit
│   ├── improvement-plan.md                 ← plán optimalizace
│   ├── testing-guide.md                    ← jak testovat
│   ├── lighthouse-baseline.report.html     ← desktop baseline report
│   ├── lighthouse-baseline.report.json
│   ├── lighthouse-baseline-mobile          ← mobile baseline JSON
│   ├── (další lighthouse reporty...)
│   └── prezentace/
│       ├── 01-prehled-optimalizace.md      ← přehled celé optimalizace
│       ├── 02-technicka-dokumentace.md     ← technický deep-dive
│       ├── 03-nastroje-a-vysledky.md       ← nástroje a výsledky
│       ├── 04-body-prezentace.md           ← talking points + rozdělení práce
│       ├── 05-pred-a-po.md                 ← before/after srovnání
│       ├── index-komentovany.html          ← anotovaná kopie s CZ komentáři
│       ├── prezentace.html                 ← interaktivní prezentační web
│       └── chat-log.md                     ← TENTO SOUBOR
```

---

## Uživatelovy požadavky (přesné citace)

1. "now i want you to focus on documenting all things done what you did and this whole chat archived. please save everything into some structured folder as files .md you can read."

2. "then please make a copy version of the index html but with comments explaining everything - what is it, how it works, why its there etc.. we will probably showcase some of the code, so better to make the comments in Czech language."

3. "then some other file that would be just text file explaining in pure words how things work, the structure, the load time priority by milliseconds in order, then the important code snippet parts explained detaildly on how actually works, some overall summary at the end."

4. "Do not edit the code without me confirming. Do everything."

5. "Now can you make just another website with only focus on the content of it and the frontend, ui. Would be good if we could visualize it, not just have all the documentation in .md or .txt but actually you can make good website as the presentation."

6. "We also should talk about as first thing that we randomly picked our client business identity and followed it through to truly simulate the real worlds process. please include this as whole first new section."

7. "save copy of this whole chat somewhere"
