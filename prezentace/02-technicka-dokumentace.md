# Technická dokumentace — SmartHome.cz

## Struktura souboru index.html

> **Pozn.:** Strom a čísla řádků níže popisují původní (Fáze 1) verzi webu o 1020 řádcích. Po rozšíření (Fáze 5) má `index.html` **1690 řádků** a navíc obsahuje: responzivní navigaci s hamburgerem (`#nav-toggle` + `#primary-nav`, ~ř. 794–810), galerii (`#galerie`, ~ř. 903), sekci produktů (`#produkty`, ~ř. 1071) a payment wall modal (`#paywall`, ~ř. 1249). Tyto nové části jsou technicky rozebrané v částech **11–13** na konci tohoto dokumentu.

```
index.html (1020 řádků — Fáze 1)
│
├── <head> (řádky 1–528)
│   ├── Meta tagy (1–11)
│   ├── Preload & preconnect (13–22)
│   ├── Asynchronní fonty (24–28)
│   ├── Deferred skripty — GSAP (30–33)
│   └── <style> blok (35–527)
│       ├── Tailwind preflight reset (37–45)
│       ├── Utility třídy (~190 tříd) (47–262)
│       ├── Custom CSS proměnné (264–268)
│       ├── Glassmorphism styly (270–329)
│       ├── Button liquid metal efekt (331–420)
│       ├── Custom cursor (422–442)
│       ├── Metalický obrázek filtry (454–463)
│       ├── Content-visibility optimalizace (469–473)
│       ├── CSS animace (475–486)
│       ├── Skip link + přístupnost (488–522)
│       └── GSAP nav transform fix (524–526)
│
├── <body> (řádky 530–1019)
│   ├── Skip link (533)
│   ├── Noise overlay (536)
│   ├── Liquid canvas (537)
│   ├── Custom cursor (540)
│   ├── Glassmorphism navigace (543–557)
│   ├── <main> hero sekce (560–645)
│   │   ├── Badge „Architektura chytrého bydlení" (566–570)
│   │   ├── <h1> s reveal animací (573–584)
│   │   ├── Popis text s CSS animací (586–589)
│   │   ├── CTA tlačítko „Konzultace" (591–595)
│   │   └── Hero obrázek + widgety (599–642)
│   ├── <section> features (648–731)
│   │   ├── Shuffler karta (654–665)
│   │   ├── Typewriter karta (668–688)
│   │   └── Cursor scheduler karta (691–728)
│   ├── <footer> (734–782)
│   └── <script> inline JS (784–1018)
│       ├── Mousemove handler + parallax (786–827)
│       ├── GSAP timeline (830–874)
│       ├── Shuffler logika (879–914)
│       ├── Typewriter efekt (917–928)
│       ├── Cursor scheduler GSAP (933–948)
│       └── Paper.js conditional loading (952–1017)
```

---

## Pořadí načítání (Load Order) — po milisekundách

Následuje přesný waterfall jak prohlížeč zpracovává stránku:

### 0ms — HTML parsing začíná
```
Browser stáhne index.html → začne parsovat <head>
```

### ~1ms — Meta tagy zpracovány
```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="description" content="...">
<meta name="theme-color" content="#f1f3f5">
<link rel="icon" href="data:image/svg+xml,...">  ← inline SVG favicon, žádný HTTP požadavek
```

### ~2ms — Preload a preconnect odeslány (paralelně)
```
→ DNS lookup + TCP connect k vekaprod-media.e-spirit.cloud (preconnect)
→ DNS lookup + TCP connect k fonts.googleapis.com (preconnect)
→ DNS lookup + TCP connect k fonts.gstatic.com (preconnect)
→ DNS prefetch k cdnjs.cloudflare.com
→ Fetch hero obrázek zahájeno (preload, fetchpriority="high")
```

**Proč je toto důležité:**
Preconnect ušetří ~100–200ms na každém doménovém spojení (DNS + TCP + TLS handshake). Bez preconnectu by prohlížeč čekal až narazí na `<img>` element v HTML.

### ~3ms — Fonty (asynchronně, NEBLOKUJÍ render)
```html
<link rel="stylesheet" href="fonts.googleapis.com/..." media="print" onload="this.media='all'">
```

**Jak to funguje:**
1. `media="print"` → prohlížeč stáhne stylesheet, ale NEBLOKUJE render (print stylesheets nejsou render-critical)
2. Po stažení se spustí `onload="this.media='all'"` → stylesheet se aktivuje pro všechna média
3. `<noscript>` fallback → pokud je JS vypnutý, fonty se načtou klasicky (blocking)
4. Výsledek: text se zobrazí okamžitě se systémovým fontem, pak se přepne na web font (FOUT — Flash of Unstyled Text)

**Úspora: ~1458ms na mobilu** (dříve render-blocking)

### ~4ms — GSAP skripty (deferred, NEBLOKUJÍ render)
```html
<script src="gsap.min.js" defer></script>
<script src="ScrollTrigger.min.js" defer></script>
```

**`defer` znamená:**
1. Stahování probíhá PARALELNĚ s HTML parsingem
2. Spuštění se odloží až PO úplném naparsování HTML
3. Zachovává pořadí — ScrollTrigger se spustí až po GSAP core

### ~5ms–~50ms — Inline CSS zpracování
```
Browser parsuje <style> blok (527 řádků CSS)
→ Vytvoří CSSOM (CSS Object Model)
→ Žádný síťový požadavek — CSS je inline = okamžitě dostupné
```

**Toto je klíčová optimalizace:**
Původně Tailwind CDN vyžadoval:
1. Stáhnout 127KB JS (síťový požadavek)
2. Spustit JIT kompilaci (CPU čas)
3. Vygenerovat CSS a vložit do DOM
4. Teprve pak mohl prohlížeč začít kreslit

Nyní: CSS je přímo v HTML → žádné čekání.

### ~50ms — First Contentful Paint (FCP) na desktopu
```
Browser má HTML + CSS → může nakreslit první pixel
→ Vykreslí text, pozadí, navigaci (se systémovým fontem)
→ Hero text animace začíná (CSS @keyframes fadeSlideUp)
```

### ~100ms — Hero obrázek dorazí (preloaded)
```
Obrázek 540×405px (72KB JPEG) byl prefetchnut už od ~2ms
→ LCP element na desktopu se vykreslí
```

### ~200ms–500ms — Web fonty dorazí
```
Inter + Syne woff2 soubory staženy
→ font-display: swap → text se přepne ze systémového fontu
→ Krátký FOUT (Flash of Unstyled Text) — akceptovatelné
```

### ~500ms — DOMContentLoaded
```
HTML plně naparsováno
→ defer skripty (GSAP, ScrollTrigger) se spustí v pořadí
```

### ~600ms–1000ms — window.load event
```
Všechny zdroje staženy (obrázky, fonty, skripty)
→ GSAP timeline startuje:
  0.2s: Navigace fade-in + slide down
  0.3s: gsap-fade elementy (badge, CTA button)
  0.4s: Reveal text animace (h1 skewY)
  0.8s: Hero obrázek slide-in
  1.5s: Floating widgety fade-in
→ ScrollTrigger registrován
→ Shuffler, Typewriter, Cursor scheduler spuštěny
→ Paper.js (pouze desktop >1024px) dynamicky načten
```

### ~1000ms–2000ms — LCP na mobilu (throttled 4G)
```
Na mobilu (simulovaná 4G síť, 10Mbps):
→ HTML stažen ~200ms
→ CSS zpracován ~300ms
→ LCP element = <p> text (hero obrázek je hidden na mobilu)
→ Text se vykreslí s CSS animací (bez čekání na GSAP)
→ LCP ~2.0s
```

---

## Klíčové kódové části — detailní vysvětlení

### 1. Tailwind CDN nahrazení (řádky 35–262)

**Původní kód (PŘED):**
```html
<script src="https://cdn.tailwindcss.com"></script>
<script>
tailwind.config = {
    theme: {
        extend: {
            colors: { metal: { ... } },
            fontFamily: { syne: ['Syne'], inter: ['Inter'] }
        }
    }
}
</script>
```

**Nový kód (PO):**
```css
<style>
    /* Preflight reset (Tailwind default) */
    *, *::before, *::after { box-sizing: border-box; ... }

    /* ~190 utility tříd ručně extrahovaných */
    .relative { position: relative; }
    .flex { display: flex; }
    .text-metal-900 { color: #212529; }
    /* ... */
</style>
```

**Proč to funguje:**
- Tailwind CDN je JIT kompilátor v prohlížeči — skenuje DOM, generuje CSS za běhu
- Pro produkci je správný postup: extrahovat jen použité třídy jako čisté CSS
- Bez build systému (CLI `npx tailwindcss`) jsme to udělali ručně — prošli jsme všechny `class=""` atributy v HTML a napsali odpovídající CSS pravidla

**Riziko:** Pokud JS dynamicky přidává třídy (shuffler karty), musíme je zahrnout ručně. Shuffler používá: `bg-white/70 backdrop-blur-md border border-metal-200 p-6 rounded-[2rem] shadow-sm transition-all duration-700 ease-[cubic-bezier(...)]`

---

### 2. Asynchronní načítání fontů (řádky 24–28)

```html
<link rel="stylesheet"
    href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Syne:wght@600;700;800&display=swap"
    media="print" onload="this.media='all'">
<noscript>
    <link rel="stylesheet" href="...stejná URL...">
</noscript>
```

**Mechanismus:**
1. `media="print"` — prohlížeč ví, že tento stylesheet je jen pro tisk → nestahuje ho jako render-critical
2. ALE stále ho stáhne (s nízkou prioritou) — prohlížeče stahují i print stylesheety
3. `onload="this.media='all'"` — jakmile se stylesheet stáhne, JS změní media query na `all` → stylesheet se aplikuje
4. `<noscript>` — fallback pro prohlížeče bez JS (např. screen readery, archivy)

**Výsledek:** Místo 1458ms render-blocking čekání → text se zobrazí okamžitě, fonty se načtou na pozadí

---

### 3. CSS-only animace pro LCP element (řádky 480–486)

```css
@keyframes fadeSlideUp {
    from { opacity: 0; transform: translateY(15px); }
    to { opacity: 1; transform: translateY(0); }
}
.hero-text-enter {
    animation: fadeSlideUp 1s cubic-bezier(0.16, 1, 0.3, 1) both;
}
```

**Proč ne GSAP?**
Původně měl hero text třídu `opacity-0 gsap-fade` — čekal na GSAP, který se spouští na `window.load`. Na throttled mobilu to znamenalo čekání 3–5 sekund (dokud se nenačtou všechny zdroje). Prohlížeč nemohl spočítat LCP, protože element měl `opacity: 0`.

Řešení: CSS animace začíná OKAMŽITĚ po zpracování `<style>` bloku — žádné čekání na JS.

---

### 4. Transform composition bug (řádek 524–526)

```css
.gsap-nav {
    transform: translateX(-50%) translateY(-20px);
}
```

**Příběh:**
Navigace je centrovaná pomocí `left: 50%` + `transform: translateX(-50%)`. GSAP při `window.load` animuje `translateY` na 0.

Problém nastal po odstranění Tailwind CDN: Tailwind injektoval styly PŘED custom CSS → `translateX(-50%)` přepisoval custom CSS. Po přechodu na inline CSS je cascade pořadí jiné — custom CSS `.gsap-nav { transform: translateY(-20px) }` PŘEPSAL Tailwind utility `.-translate-x-1/2` → navigace přestala být centrovaná (přetekla vpravo).

Oprava: Kombinovat oba transformy v jednom deklaraci: `translateX(-50%) translateY(-20px)`

---

### 5. Podmíněné načítání Paper.js (řádky 952–1017)

```javascript
if (window.innerWidth > 1024) {
    const paperScript = document.createElement('script');
    paperScript.src = 'https://cdnjs.cloudflare.com/ajax/libs/paper.js/0.12.17/paper-full.min.js';
    paperScript.onload = function () {
        paper.install(window);
        paper.setup('liquid-canvas');
        // ... blob setup ...
    };
    document.head.appendChild(paperScript);
}
```

**Proč:**
- Paper.js má 67KB (transfer) / 69KB (parsed)
- Na mobilu je tekutý blob efekt na pozadí sotva viditelný
- Šetříme 67KB bandwidth + CPU čas na parsování a spouštění
- `document.createElement('script')` — dynamicky injektovaný skript je defaultně `async` → neblokuje

---

### 6. Content-visibility optimalizace (řádky 469–473)

```css
#features, footer {
    content-visibility: auto;
    contain-intrinsic-size: auto 800px;
}
```

**Co to dělá:**
- `content-visibility: auto` — prohlížeč přeskočí rendering elementů, které nejsou ve viewportu
- `contain-intrinsic-size: auto 800px` — rezervuje ~800px výšky, aby se CLS (layout shift) neprojevil při scrollování
- Šetří CPU čas při initial paint — features sekce a footer se nevykreslují dokud uživatel nescrolluje

---

### 7. Glassmorphism efekt (řádky 264–322)

```css
:root {
    --glass-bg: rgba(255, 255, 255, 0.25);
    --glass-border: rgba(255, 255, 255, 0.4);
    --blur: blur(24px);
}

.glass-card {
    background: rgba(255, 255, 255, 0.35);
    backdrop-filter: var(--blur);
    -webkit-backdrop-filter: var(--blur);
    border: 1px solid var(--glass-border);
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.05), inset 0 1px 0 rgba(255, 255, 255, 0.9);
    border-radius: 24px;
}
```

**Jak glassmorphism funguje:**
1. Poloprůhledné pozadí (`rgba(255, 255, 255, 0.35)`) — karty jsou 35% neprůhledné
2. `backdrop-filter: blur(24px)` — rozostří obsah ZA kartou
3. Bílý border (`rgba(255, 255, 255, 0.4)`) — jemný okraj simulující sklo
4. Inset box-shadow — horní highlight simulující odraz světla na skle
5. `-webkit-backdrop-filter` — vendor prefix pro Safari

---

### 8. Liquid Metal tlačítko (řádky 331–420)

```css
@property --angle {
    syntax: '<angle>';
    initial-value: 0deg;
    inherits: false;
}

.btn-liquid-metal::before {
    background: conic-gradient(from var(--angle), #fff 0%, #dee2e6 15%, ...);
    animation: spin-metallic 4s linear infinite;
}

@keyframes spin-metallic {
    to { --angle: 360deg; }
}
```

**Jak to funguje:**
1. `@property --angle` registruje CSS custom property s typem `<angle>` — to umožňuje animovat ji
2. `conic-gradient(from var(--angle), ...)` — kuželový gradient rotující kolem středu
3. `::before` pseudo-element je o 2px větší než tlačítko (top/left: -2px) — vytváří animovaný okraj
4. `::after` pseudo-element vyplňuje vnitřek bílým gradientem — překrývá rotující gradient uprostřed
5. Výsledek: metalický shimmer efekt na okraji tlačítka

**W3C validátor hlásí chybu** na `@property` a `conic-gradient(var())` — ale je to **validní moderní CSS** (Chrome 78+, Safari 16.4+). Validátor je zastaralý.

---

### 9. GSAP Timeline (řádky 830–874)

```javascript
window.addEventListener('load', () => {
    gsap.registerPlugin(ScrollTrigger);
    const tl = gsap.timeline({ defaults: { ease: "power4.out" } });

    // 0.2s: Navigace slide-down + fade-in
    tl.to('.gsap-nav', { y: 0, opacity: 1, duration: 1.5, top: "2rem" }, 0.2);

    // 0.4s: H1 text reveal s skewY efektem
    tl.fromTo('.reveal-text', { y: "150%", skewY: 15 }, { y: "0%", skewY: 0, duration: 1.6, stagger: 0.15 }, 0.4);

    // 0.3s: Badge + CTA fade-in (offset 0.3 místo 1.0 pro rychlejší start)
    tl.to('.gsap-fade', { y: 0, opacity: 1, duration: 1.2, stagger: 0.2, y: -15 }, 0.3);

    // 0.8s: Hero obrázek entrance
    tl.to('.gsap-img', { opacity: 1, scale: 1, duration: 1.8 }, 0.8);

    // 1.5s: Floating widgety
    tl.to('.gsap-widget', { opacity: 1, duration: 1.2, stagger: 0.3 }, 1.5);
});
```

**Proč `window.load` a ne `DOMContentLoaded`?**
GSAP skripty jsou `defer` — spouští se na `DOMContentLoaded`. Ale `paper.install(window)` na starší verzi Paper.js potřebuje globální scope. Proto čekáme na `load` — zaručuje, že GSAP + ScrollTrigger jsou plně inicializovány.

---

### 10. Přístupnost (Accessibility)

```html
<!-- Skip link — první element v <body> -->
<a href="#main-content" class="skip-link">Přeskočit na obsah</a>

<!-- Main landmark s ID pro skip link -->
<main id="main-content" class="...">
```

```css
/* Skrytý link, který se zobrazí na focus (Tab klávesa) */
.skip-link {
    position: absolute;
    top: -100%;  /* skrytý nad viewport */
    z-index: 10000;
}
.skip-link:focus {
    top: 0;  /* zobrazí se na focus */
}

/* Focus styly pro klávesovou navigaci */
a:focus-visible, .btn-liquid-metal:focus-visible {
    outline: 2px solid #495057;
    outline-offset: 4px;
}

/* Respektování uživatelského nastavení pro animace */
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
    }
}
```

**WCAG 2.1 AA požadavky splněné:**
1. **Skip link** — uživatelé klávesnice mohou přeskočit navigaci
2. **Focus-visible** — viditelné outlines při Tab navigaci (ne při kliknutí)
3. **Reduced motion** — pokud má OS nastaveno „reduce motion", animace se vypnou
4. **aria-label** na `<nav>` — screen readery mohou identifikovat navigaci
5. **Sémantické HTML** — `<main>`, `<nav>`, `<section>`, `<footer>` jako ARIA landmarks

---

## Rozšíření webu (Fáze 5) — technické detaily

### 11. Responzivní obrázky — galerie a produkty (`#galerie` ~ř. 903, `#produkty` ~ř. 1071)

Každý obrázek v galerii i u produktů je `<picture>` se dvěma moderními formáty a více velikostmi:

```html
<picture>
  <source type="image/avif"
          srcset="img/g1-400.avif 400w, img/g1-800.avif 800w, img/g1-1200.avif 1200w"
          sizes="(max-width: 768px) 100vw, (max-width: 1100px) 50vw, 33vw">
  <source type="image/webp"
          srcset="img/g1-400.webp 400w, img/g1-800.webp 800w, img/g1-1200.webp 1200w"
          sizes="(max-width: 768px) 100vw, (max-width: 1100px) 50vw, 33vw">
  <img src="img/g1-800.webp" alt="..." width="800" height="600"
       loading="lazy" decoding="async">
</picture>
```

**Jak to funguje:**
1. `<source type="image/avif">` — prohlížeč s podporou AVIF (nejmenší formát) si vezme tento zdroj
2. `<source type="image/webp">` — fallback pro prohlížeče bez AVIF
3. `<img>` — finální fallback (a nositel `alt`, `width`, `height`, `loading`)
4. `srcset` se šířkovými deskriptory (`400w/800w/1200w`) + `sizes` → prohlížeč spočítá, kterou velikost reálně potřebuje pro daný viewport a DPR, a stáhne **jen tu jednu**
5. `loading="lazy"` — obrázky pod foldem se nestahují, dokud k nim uživatel nedoscrolluje
6. `width` + `height` (a v CSS `aspect-ratio`) → prohlížeč rezervuje místo předem → **CLS = 0**

**Generování variant:**
```bash
# WebP (q80 galerie, q82 produkty)
cwebp -q 80 zdroj.jpg -o g1-800.webp
# AVIF (q58 galerie, q60 produkty)
avifenc --min 0 --max 63 -q 58 zdroj.jpg g1-800.avif
```
Celkem **44 variant**: galerie 4 obrázky × 3 velikosti × 2 formáty (24) + produkty 5 obrázků × 2 velikosti × 2 formáty (20).

**Proč AVIF + WebP:** AVIF je typicky o 30–50 % menší než JPEG při stejné kvalitě, WebP o 25–35 %. Dva formáty v `<picture>` pokryjí prakticky všechny moderní prohlížeče bez zhoršení výkonu.

---

### 12. Responzivní hamburger menu (`#nav-toggle` + `#primary-nav`, ~ř. 794–810)

```html
<button id="nav-toggle" class="nav-toggle" aria-expanded="false"
        aria-controls="primary-nav" aria-label="Otevřít menu">
  <span></span><span></span><span></span>
</button>
<div id="primary-nav"> ...odkazy... </div>
```

```javascript
// Samostatné IIFE — nezávisí na načtení GSAP
(function () {
  const toggle = document.getElementById('nav-toggle');
  const nav = document.getElementById('primary-nav');
  toggle.addEventListener('click', () => {
    const open = toggle.getAttribute('aria-expanded') === 'true';
    toggle.setAttribute('aria-expanded', String(!open));
    nav.classList.toggle('is-open');
  });
  // zavření klávesou ESC
  document.addEventListener('keydown', e => {
    if (e.key === 'Escape') { /* zavřít */ }
  });
})();
```

**Klíčové body:**
- Pod **768px** se desktop odkazy schovají a zobrazí se hamburger ikona (3 čárky → animují se na „X")
- **Přístupnost:** `aria-expanded` se přepíná, `aria-controls` propojuje tlačítko s menu, `aria-label` popisuje akci
- JS je v **samostatném IIFE**, takže menu funguje, i kdyby se GSAP nenačetl
- Ovládání klávesnicí + zavření přes ESC

---

### 13. Payment wall — modální okno (`#paywall`, ~ř. 1249)

```html
<div class="paywall" id="paywall" role="dialog" aria-modal="true"
     aria-labelledby="paywall-title" aria-hidden="true">
  <div class="paywall-card"> ...formulář... </div>
</div>
```

Spouštěče (hero „Konzultace", footer „Nezávazná poptávka", tlačítka „Objednat" u produktů):
```html
<button data-checkout data-item="tado Chytrý termostat V3+"
        data-price="2 490 Kč">Objednat</button>
```

**Mechanismus:**
1. Klik na libovolný prvek s `data-checkout` otevře modal a předvyplní název položky + cenu
2. **Focus trap** — Tab cykluje jen uvnitř modalu (přístupnost)
3. Zavření: klávesa **ESC** nebo klik na ztmavené pozadí
4. `aria-modal="true"` + `aria-hidden` toggle → screen readery vědí, že zbytek stránky je dočasně mimo
5. Formulář: jméno, e-mail, číslo karty (auto-formát po čtveřicích regexem `(.{4})(?=.)`), expirace, CVC → po odeslání success obrazovka
6. JS je opět **samostatné IIFE** nezávislé na GSAP
