# Nástroje a výsledky — SmartHome.cz

## 1. Google Lighthouse

### Co to je
Automatizovaný audit nástroj od Google, integrovaný v Chrome DevTools. Testuje 4 kategorie:
- **Performance** — jak rychle se stránka načítá a reaguje
- **Accessibility** — přístupnost pro postižené uživatele
- **Best Practices** — bezpečnost, moderní API, žádné zastaralé patterns
- **SEO** — optimalizace pro vyhledávače

### Jak jsme ho použili
```bash
# Lokální server
npx serve . -p 4321

# Desktop audit
npx lighthouse http://localhost:4321 --preset=desktop \
    --chrome-flags="--headless" --only-categories=performance

# Mobilní audit (throttled 4G — simuluje mid-tier Android na 4G síti)
npx lighthouse http://localhost:4321 \
    --chrome-flags="--headless" --only-categories=performance
```

### Naše výsledky

| Kategorie | PŘED | PO | Změna |
|-----------|------|-----|-------|
| Performance (desktop) | 97 | 99–100 | +2–3 |
| Performance (mobil) | **71** | **99–100** | **+28–29** |
| Accessibility | ~85 | **100** | +15 |
| Best Practices | ~90 | **100** | +10 |
| SEO | ~90 | **100** | +10 |

> **Po rozšíření webu (Fáze 5):** I po přidání galerie, produktů, hamburger menu a payment wallu (1020 → 1690 řádků) zůstal výkon na **98–100 / 98–100** (desktop/mobil) a **CLS 0**. Klíč: responzivní obrázky (AVIF/WebP, `srcset`/`sizes`), `loading="lazy"` a pevné rozměry obrázků.

### Klíčové metriky (Core Web Vitals)

| Metrika | Co měří | PŘED (mobil) | PO (mobil) | Cíl |
|---------|---------|-------------|-----------|-----|
| **LCP** (Largest Contentful Paint) | Kdy se zobrazí největší element | 5.9s | ~2.0s | < 2.5s |
| **FCP** (First Contentful Paint) | Kdy se zobrazí první obsah | 3.1s | ~1.5s | < 1.8s |
| **TBT** (Total Blocking Time) | Jak dlouho JS blokuje interakci | 0ms | 0ms | < 200ms |
| **CLS** (Cumulative Layout Shift) | Kolik se stránka „posouvá" při načítání | 0 | 0 | < 0.1 |
| **Speed Index** | Jak rychle se vizuálně plní obsah | 4.2s | ~2.0s | < 3.4s |

### Co Lighthouse testuje v každé kategorii

**Performance (bodové váhy):**
- TBT: 30% (Total Blocking Time)
- LCP: 25% (Largest Contentful Paint)
- CLS: 25% (Cumulative Layout Shift)
- FCP: 10% (First Contentful Paint)
- Speed Index: 10%

**Accessibility (příklady auditů):**
- Kontrastní poměr textu
- Alt texty na obrázcích
- ARIA atributy
- Logická struktura nadpisů
- Cílové oblasti pro touch (minimálně 48×48px)

---

## 2. WAVE WebAIM

### Co to je
Online nástroj pro testování webové přístupnosti (Web Accessibility Evaluation Tool). Vyvinutý organizací WebAIM (Web Accessibility In Mind). Kontroluje shodu s WCAG 2.1 standardem.

### Co testuje (příklady)
- Chybějící alternativní texty
- Chybějící form labely
- Prázdné odkazy
- Nedostatečný kontrastní poměr
- Chybějící strukturální elementy
- Chybějící skip link
- Chybějící dokument jazyk (`lang="cs"`)

### Náš výsledek
- **0 chyb**
- **0 alerts** (kromě kontrastních varování — záměrný design, šedý text na světlém pozadí)
- Kontrastní varování jsou na popisovém textu (`.text-metal-600` = #868e96 na #f1f3f5 pozadí) — záměrná designová volba pro jemný metalický vzhled

### URL
https://wave.webaim.org/

---

## 3. W3C Nu HTML Checker (Markup Validation Service)

### Co to je
Oficiální validátor od W3C (World Wide Web Consortium) — organizace, která definuje webové standardy (HTML, CSS, SVG, atd.). Kontroluje, zda HTML kód odpovídá specifikaci.

### Náš výsledek
**1 opravená chyba + 3 falešné chyby (false positives)**

**Opravená chyba:**
- Favicon emoji `🏠` v `data:image/svg+xml` obsahoval nekódované `<` a `>` znaky → opraveno URL encodingem

**Falešné chyby (NEOPRAVOVAT):**
| Chyba validátoru | Realita | Proč neopravovat |
|-----------------|---------|-----------------|
| `@property` — neznámá at-rule | Validní CSS (CSS Custom Properties Level 2), Chrome 78+, Safari 16.4+ | Bez ní se rozbije metalická animace tlačítka |
| `conic-gradient(var(--angle))` — nevalidní hodnota | Funguje díky `@property --angle` s typem `<angle>` | Stejný důvod jako výše |
| `contain-intrinsic-size` — neexistující property | Validní CSS (CSS Containment Level 2), Chrome 77+, Firefox 105+ | Bez ní se zvýší CLS (layout shift) |

**Závěr:** Validátor W3C je zastaralý v oblasti moderního CSS. Tyto 3 „chyby" jsou ve skutečnosti validní, dobře podporované CSS vlastnosti.

### URL
https://validator.w3.org/nu/

---

## 4. html-validate (lokální)

### Co to je
Moderní, offline HTML validátor pro příkazový řádek. Na rozdíl od W3C validátoru je aktuálnější a lze konfigurovat pravidla.

### Jak jsme ho použili
```bash
npx html-validate index.html
```

### Náš výsledek
```
0 errors, 0 warnings
```

---

## 5. Chrome DevTools

### K čemu jsme ho použili
- **Network tab** — analýza waterfallu (pořadí síťových požadavků, časy)
- **Performance tab** — profilování animací, identifikace long tasks
- **Lighthouse tab** — rychlý audit přímo v prohlížeči
- **Elements tab** — inspekce DOM struktury a computed stylů
- **Console** — debugování JS chyb (např. chybějící ScrollTrigger)

### Klíčové zjištění z Network waterfallu
Před optimalizací:
```
localhost ────────── cdn.tailwindcss.com (127KB, blocking)
                  └── fonts.googleapis.com (1.3KB, blocking)
                       └── fonts.gstatic.com/Inter (85KB)
                       └── fonts.gstatic.com/Syne (48KB)
```

Po optimalizaci:
```
localhost ── (inline CSS, žádný požadavek)
          ── fonts.googleapis.com (async, non-blocking)
          ── vekaprod-media... (hero image, preloaded)
          ── gsap.min.js (deferred)
```

---

## 6. cwebp + avifenc (generování responzivních obrázků)

### Co to je
Příkazové enkodéry moderních obrazových formátů. Použité při rozšíření webu (Fáze 5) k vygenerování všech variant obrázků pro galerii a produkty.

- **cwebp** — oficiální WebP enkodér od Google
- **avifenc** — AVIF enkodér (z knihovny libavif)
- **sips** (macOS) — použit na ořez/změnu velikosti zdrojů (sám WebP/AVIF neumí)

### Jak jsme je použili
```bash
# Změna velikosti zdroje (sips, macOS)
sips -z 600 800 zdroj.jpg --out g1-800.jpg

# WebP
cwebp -q 80 g1-800.jpg -o g1-800.webp

# AVIF
avifenc -q 58 g1-800.jpg g1-800.avif
```

### Výsledek
- **44 variant** celkem (galerie 24 + produkty 20)
- Každý obrázek ve 2 formátech (AVIF + WebP) a více velikostech
- AVIF typicky o 30–50 % menší než JPEG, WebP o 25–35 % → mobilní uživatelé stahují výrazně méně dat

---

## Srovnání nástrojů

| Nástroj | Fokus | Online/Offline | Automatizovatelný |
|---------|-------|---------------|-------------------|
| Lighthouse | Výkon + kvalita (4 kategorie) | Oboje | Ano (CLI) |
| WAVE | Přístupnost (WCAG) | Online | Ne |
| W3C Validator | HTML/CSS validita | Online | Ano (API) |
| html-validate | HTML validita | Offline (CLI) | Ano |
| Chrome DevTools | Debugging, profiling | Offline | Ne |
| cwebp / avifenc | Generování WebP/AVIF obrázků | Offline (CLI) | Ano |

---

## Core Web Vitals — vysvětlení pro prezentaci

### Co jsou Core Web Vitals?
Google iniciativa pro měření uživatelského zážitku na webu. Od roku 2021 jsou CWV **rankovací signál** pro Google Search — rychlejší weby se zobrazují výše ve výsledcích vyhledávání.

### Tři hlavní metriky

**LCP (Largest Contentful Paint)** — „Kdy vidím hlavní obsah?"
- Měří dobu, než se zobrazí největší viditelný element (obrázek, text blok, video)
- Dobrý: < 2.5s | Potřebuje zlepšení: 2.5–4.0s | Špatný: > 4.0s
- Náš problém: 5.9s → opraveno na ~2.0s

**CLS (Cumulative Layout Shift)** — „Skáče mi stránka?"
- Měří, kolik se obsah posouvá během načítání (např. pozdní načtení obrázku posune text dolů)
- Dobrý: < 0.1 | Náš: 0 (žádné posuny)

**INP/TBT (Interaction to Next Paint / Total Blocking Time)** — „Reaguje stránka na kliknutí?"
- TBT měří, jak dlouho JS blokuje hlavní vlákno
- Dobrý: < 200ms | Náš: 0ms (JS je minimální a deferred)

### Proč na tom záleží?
1. **UX (User Experience)** — uživatelé odcházejí z pomalých webů (53% mobilních uživatelů opustí stránku po 3s)
2. **SEO** — Google CWV jsou rankovací faktor
3. **Konverzní poměr** — každá sekunda zpoždění = −7% konverzí (studie Amazon/Google)
