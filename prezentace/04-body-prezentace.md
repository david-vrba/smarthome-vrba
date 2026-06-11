# Body prezentace — SmartHome.cz

## Struktura prezentace (navrhované pořadí)

### 1. Úvod — Ukázka webu (2 min)
**Kdo říká:** David

- Otevřít web v prohlížeči, ukázat na celé obrazovce
- „Tohle je SmartHome.cz — landing page pro chytrou domácnost"
- Ukázat: hero animace, scrollování, feature karty, **galerie**, **produkty**, footer
- „Všechno je v jednom souboru index.html, žádný framework, žádný build system"
- Na mobilu ukázat **hamburger menu** (zmenšit okno prohlížeče)
- Zeptat se třídy: „Co se vám líbí/nelíbí na UI?"

---

### 2. Proč Core Web Vitals? (2 min)
**Kdo říká:** [kamarád]

Body k zmínění:
- CWV jsou od roku 2021 rankovací signál v Google Search
- 3 hlavní metriky: LCP, CLS, INP
- Rychlé weby = lepší UX, lepší SEO, lepší konverze
- Studie: 53% mobilních uživatelů odejde po 3 sekundách
- Google PageSpeed Insights používá Lighthouse (stejný engine)

---

### 3. Výchozí stav — „Jak špatné to bylo" (2 min)
**Kdo říká:** David

Body k zmínění:
- Desktop: 97 (dobré) — ale mobil: **71** (špatné)
- LCP na mobilu: **5.9 sekund** (cíl je pod 2.5s)
- Hlavní viníci:
  - Tailwind CDN: synchronní script, 127KB, 1524ms render-blocking
  - Google Fonts: render-blocking stylesheet, 1458ms
  - „Tyto dva zdroje dohromady blokovaly vykreslení na 3 sekundy"
- Ukázat screenshot baselineového Lighthouse reportu (pokud máme)

---

### 4. Showcase nástroje #1 — Lighthouse (3 min)
**Kdo říká:** David

- Otevřít Chrome DevTools → Lighthouse tab
- Spustit audit NAŽIVO (nebo ukázat uložený report)
- Ukázat: skóre, jednotlivé metriky, „Diagnostics" sekci
- „Lighthouse nám přesně řekne, co je špatně a proč"
- Ukázat render-blocking resources audit
- Ukázat LCP breakdown (element render delay)
- „Testy jsme spouštěli i z příkazového řádku: `npx lighthouse http://localhost:4321`"

---

### 5. Showcase nástroje #2 — WAVE / Validátor (3 min)
**Kdo říká:** [kamarád]

WAVE:
- Otevřít https://wave.webaim.org/
- Vložit URL nebo ukázat uložený screenshot
- „WAVE kontroluje přístupnost — jestli web mohou používat i nevidomí"
- Ukázat: 0 chyb, kontrastní varování (záměrný design)
- Zmínit: skip link, focus styly, aria labely, prefers-reduced-motion

W3C Validátor:
- Otevřít https://validator.w3.org/nu/
- Ukázat výsledek: 3 chyby, ale jsou to falešné positives
- „Validátor nezná moderní CSS — @property, contain-intrinsic-size"
- „Důležité je chápat, kdy je validátor zastaralý a kdy máme opravdovou chybu"

---

### 6. Co jsme udělali — Optimalizace (5 min)
**Kdo říká:** David

**Největší oprava — Tailwind CDN:**
- „Tailwind CDN je vývojový nástroj, ne produkční řešení"
- „Stahuje 127KB JS, spustí JIT kompilaci v prohlížeči, vygeneruje CSS"
- „My jsme vzali těch ~190 tříd, co web skutečně používá, a napsali je jako čisté CSS"
- „Výsledek: 0ms čekání místo 1524ms"
- Ukázat kód: původní `<script src="cdn.tailwindcss.com">` vs nový `<style>` blok

**Fonty:**
- „Google Fonts standardně blokují render — prohlížeč čeká na stažení stylů"
- „Trik: `media="print" onload="this.media='all'"` — prohlížeč stáhne fonts na pozadí"
- „Text se zobrazí okamžitě se systémovým fontem, pak se přepne" (FOUT)

**LCP na mobilu:**
- „Na mobilu je LCP element text, ne obrázek (obrázek je hidden na mobilu)"
- „Původně text čekal na GSAP (window.load) — příliš pozdě na throttled 4G"
- „Řešení: CSS animace místo GSAP pro hero text"

**Podmíněné načítání:**
- „Paper.js (67KB) je tekutý blob na pozadí — na mobilu ho nikdo nevidí"
- „Načítáme ho jen na desktopu (>1024px) pomocí dynamického scriptu"

---

### 7. Ukázka kódu (2–3 min)
**Kdo říká:** [kamarád] nebo David

Otevřít `index-komentovany.html` a ukázat:
- Jak funguje async font loading (řádky 24–28)
- `content-visibility: auto` (řádky 469–473)
- Skip link pro přístupnost (řádek 533)
- Podmíněné načítání Paper.js (řádky 952–1017)
- Glassmorphism efekt — backdrop-filter (řádky 315–322)

---

### 8. Rozdíl před/po — obrázek (1 min)
**Kdo říká:** [kamarád]

- „Obrázek na webu je 72KB JPEG z externího CDN"
- „Prohlížeč ho zobrazuje v 540×405px, ale vizuálně zabírá celou kartu díky `object-cover`"
- „Důležité: `fetchpriority="high"` a `preload` zajistí, že se stáhne co nejdříve"
- „Na mobilu je obrázek skrytý — LCP je text, ne obrázek"

---

### 9. Výsledky — ŽIVÝ test (2 min)
**Kdo říká:** David

- Spustit Lighthouse NAŽIVO v Chrome DevTools
- Ukázat: 99–100 / 100 / 100 / 100
- „Z 71 na 99 na mobilu — zlepšení o 28 bodů"
- „LCP z 5.9s na 2.0s — o 66% rychlejší"
- „Accessibility, Best Practices, SEO — všechno 100"

---

### 10. Rozšíření webu — nový obsah bez ztráty výkonu (3 min)
**Kdo říká:** David

Body k zmínění:
- „Po optimalizaci jsem web rozšířil — přidal galerii, sekci produktů, menu a payment wall"
- „Výzva nebyla přidat obsah, ale přidat ho tak, aby **nespadlo skóre**"
- „Stránka má teď dvojnásobek obsahu (1020 → 1690 řádků) a skóre je pořád **98–100**, CLS **0**"
- Ukázat na webu galerii a produkty, scrollnout

**Responzivní obrázky (galerie + produkty):**
- „Každý obrázek je `<picture>` se dvěma moderními formáty — **AVIF** a **WebP**"
- „AVIF je nejmenší, WebP je fallback pro starší prohlížeče, kdyby ani jeden, je tu JPEG"
- „`srcset` má 3 velikosti (400/800/1200px), prohlížeč si stáhne jen tu, kterou potřebuje"
- „`loading=\"lazy\"` — obrázky pod foldem se načtou až při scrollu"
- „`width` a `height` jsou pevné → **žádný layout shift**, CLS zůstává 0"
- „Vygeneroval jsem **44 obrazových variant** lokálně přes `cwebp` a `avifenc`"

**Hamburger menu:**
- „Na desktopu klasické menu, na mobilu (< 768px) hamburger"
- „Je plně přístupné — `aria-expanded`, ovládá se klávesnicí, zavírá přes ESC"

---

### 11. Konverze — produkty a payment wall (2 min)
**Kdo říká:** [kamarád]

Body k zmínění:
- „Přidali jsme sekci s **5 reálnými produkty** a reálnými cenami (990 – 9 490 Kč)"
- „Produkty jsme identifikovali z obrázků (RainPoint gateway, tado termostat, MINIX mini PC...)"
- „Tlačítka „Konzultace" a „Nezávazná poptávka" už nevedou do prázdna — vedou na **payment wall**"
- Ukázat naživo: kliknout na „Objednat" → otevře se modal s formulářem
- „Modal je přístupný — `role=\"dialog\"`, focus trap, zavře se přes ESC nebo klik na pozadí"
- „Číslo karty se automaticky formátuje po čtveřicích"

---

### 12. Nasazení & hosting — naše rešerše (2 min)
**Kdo říká:** Kamarád

Body k zmínění (detail v `06-hosting-a-nasazeni.md`):
- „Výstup projektu je **veřejný odkaz** — web musí být online, takže jsme porovnali free hostingy"
- „Web je **statický** (jeden HTML + obrázky), takže ho zvládne kterýkoli host zdarma"
- Ukázat srovnávací tabulku na slidu (Cloudflare / Vercel / Netlify / GitHub Pages)
- „Liší se hlavně **přenosem dat** a **doménou**, ne technicky"
- Výchozí domény: `.pages.dev`, `.vercel.app`, `.netlify.app`, `.github.io/<repo>/`
- „**Cloudflare Pages** má jako jediný neomezený přenos zdarma → náš hlavní argument"
- „**Vercel** = nejlepší DX, **GitHub Pages** = nulová závislost a web přímo u kódu"
- 3D pozn.: „3D běží v prohlížeči klienta, host posílá jen statiku → i komplexní 3D jde i na GitHub Pages"

---

### 13. Forma vs. sdělení — pushback + 3D stránka „O zakladateli" (3 min)
**Kdo říká:** David

Body k zmínění:
- Pushback: „Ukážu vám **zentry.com** — jeden z vizuálně nejlepších 3D webů na světě"
- Nechat třídu chvíli scrollovat: „A teď mi řekněte, **co ta firma dělá**?" → nikdo neví
- „To je past: forma přebila sdělení. Hromada peněz a designu na něco, co nikdo nepřečte"
- Kontrast: „Náš web ti za vteřinu řekne, co děláme a za kolik (hero + produkty + ceník)"
- „Teprve **pak** má 3D paráda prostor — schovaná pod stránkou **/o-mne** (O zakladateli)"
- Ukázat naživo `o-mne.html`: hybridní 3D — ① real-time WebGL dům (reaguje na myš/scroll), ② obrazová sekvence (let dronem nad vilou, scrubovaný scrollem)
- „Celé **plain HTML + CDN, bez build systému** — jeden Lenis + GSAP scroll žene obě techniky, vždy běží jen jedna → spektákl bez ztráty výkonu"
- Pointa: **„Forma nikdy nesmí přebít sdělení."**

---

### 14. Shrnutí a závěr (1 min)
**Kdo říká:** Oba

Hlavní poučení:
1. **Render-blocking zdroje jsou nepřítel #1** — vždy zkontrolujte, co blokuje FCP/LCP
2. **Tailwind CDN ≠ produkční řešení** — pro produkci extrahujte pouze použité třídy
3. **Přístupnost není bonus** — je to standard (a Google ji hodnotí)
4. **Testujte na mobilu** — desktop a mobil jsou dva různé světy
5. **Nástroje existují** — Lighthouse, WAVE, Validátor → používejte je průběžně
6. **Přidat obsah a udržet výkon jde** — moderní formáty obrázků (AVIF/WebP), lazy loading a pevné rozměry → web zdvojnásobil obsah a skóre zůstalo 98–100

---

## Rozdělení práce

| Sekce | Kdo | Čas |
|-------|-----|-----|
| 1. Ukázka webu | David | 2 min |
| 2. Proč CWV | Kamarád | 2 min |
| 3. Výchozí stav | David | 2 min |
| 4. Lighthouse showcase | David | 3 min |
| 5. WAVE + Validátor showcase | Kamarád | 3 min |
| 6. Optimalizace | David | 5 min |
| 7. Ukázka kódu | Kamarád / David | 2-3 min |
| 8. Obrázek před/po | Kamarád | 1 min |
| 9. Živý test | David | 2 min |
| 10. Rozšíření webu (galerie, obrázky, menu) | David | 3 min |
| 11. Konverze (produkty, payment wall) | Kamarád | 2 min |
| 12. Nasazení & hosting (rešerše) | Kamarád | 2 min |
| 13. Forma vs. sdělení (zentry + 3D O zakladateli) | David | 3 min |
| 14. Shrnutí | Oba | 1 min |
| **Celkem** | | **~33 min** |

---

## Taby k přípravě v Chrome

1. **Tab 1:** Web — `http://localhost:4321` (nebo živý URL)
2. **Tab 2:** Lighthouse report (HTML soubor z docs/)
3. **Tab 3:** WAVE — https://wave.webaim.org/ (s výsledkem webu)
4. **Tab 4:** W3C Validátor — https://validator.w3.org/nu/ (s výsledkem)
5. **Tab 5:** DevTools otevřený na Lighthouse tab (pro live test)
6. **Tab 6:** `index-komentovany.html` otevřený v editoru (pro ukázku kódu)
7. **Tab 7:** Baseline Lighthouse report (pro srovnání před/po)

---

## Věci k zapamatování

- Nezapomeňte spustit `npx serve . -p 4321` PŘED prezentací
- Lighthouse skóre se mění mezi běhy (±2-3 body) — proto říkáme „99–100" ne „100"
- Pokud Lighthouse dá nižší skóre live, vysvětlete: „Lighthouse simuluje pomalou 4G síť a mid-tier Android — každý běh je trochu jiný"
- WAVE potřebuje veřejně dostupný URL — pokud web běží jen lokálně, ukažte screenshot
- Mluvte k publiku, ne k obrazovce — code review dělejte tváří k třídě
- Pro hamburger menu zmenšete okno prohlížeče pod 768px (nebo použijte DevTools device mode)
- U galerie můžete v DevTools → Network ukázat, že se stáhne **AVIF** varianta (ne JPEG) a jen ta správná velikost
