# Nasazení a free hosting — srovnání

Web je **statický** (jeden `index.html` + složka `img/`), takže ho zvládne hostovat zdarma libovolná platforma pro statické stránky. Výstupem projektu je **veřejný odkaz**, který pošleme učiteli — proto jsme porovnali free hostingy a vybrali podle objektivních kritérií.

---

## Jak funguje statický hosting (obecně)

Statický web = hotové soubory (HTML, CSS, JS, obrázky). Hosting je jen pošle prohlížeči přes CDN. Žádný server, žádná databáze. Veškerá logika (animace, 3D, payment wall) běží **v prohlížeči návštěvníka**. Proto je statický hosting rychlý, levný (často zdarma) a bezpečný.

---

## Jak funguje GitHub Pages (detailně)

1. Nahraješ soubory do GitHub repozitáře.
2. V **Settings → Pages** zvolíš zdroj: větev (např. `main`) a složku (`/root` nebo `/docs`), případně build přes GitHub Actions.
3. GitHub web publikuje přes HTTPS na Fastly CDN.

**Dva typy webů určují URL:**
- **User/org web:** repozitář pojmenovaný přesně `<username>.github.io` → běží na `https://<username>.github.io`
- **Project web:** jakýkoli repozitář → běží na `https://<username>.github.io/<repo>/` (pozor na podcestu)

**Limity:** ~1 GB repozitář, ~1 GB publikovaný web, **měkký limit ~100 GB přenosu/měsíc**, ~10 buildů/hod. **Pouze statika** — žádný serverový kód, SSR ani tajné API klíče.

---

## Lze na GitHub Pages nasadit i komplexní 3D web?

**Ano.** Webové 3D (Three.js, WebGL/WebGPU, Babylon.js, i React Three Fiber) běží **celé v prohlížeči na GPU návštěvníka** — hosting jen posílá statické soubory. Náročnost 3D tedy hostingem omezená není.

**Na co si dát pozor:**
- Musí to být **statický build.** Pokud použiješ R3F/Astro/Vite, build proběhne lokálně (nebo v GitHub Actions) do statického HTML/JS a ten se nahraje. Za běhu žádný Node server.
- **Velké assety** (`.glb` modely, 4K textury, HDRI) se počítají do ~1 GB repozitáře a ~100 GB přenosu — pro školní projekt v pohodě, problém až ve velkém měřítku.
- Project weby běží pod `/<repo>/`, takže cesty k assetům musí být **relativní** (nebo nastavit Vite `base`) — klasický „prázdná stránka na Pages" zádrhel.

> Závěr: i komplexní 3D web jde na GitHub Pages, host řeší jen statické soubory; jediný (měkký) limit je celková velikost assetů a přenos dat.

---

## Výchozí domény (jak vypadá adresa zdarma)

| Hosting | Výchozí URL |
|---|---|
| **Vercel** | `<projekt>.vercel.app` |
| **Netlify** | `<name>.netlify.app` (výchozí náhodný název, např. `dreamy-einstein-1a2b3c.netlify.app`, lze přejmenovat) |
| **GitHub Pages** | `<username>.github.io` nebo `<username>.github.io/<repo>/` |
| **Cloudflare Pages** | `<projekt>.pages.dev` |
| **GitLab Pages** | `<username>.gitlab.io/<projekt>` |
| **Render** | `<name>.onrender.com` |
| **Surge** | `<name>.surge.sh` |

Všechny umožňují i **vlastní doménu zdarma** (přes DNS CNAME) s automatickým HTTPS.

---

## Hlavní srovnání

| Kritérium | Cloudflare Pages | Vercel | Netlify | GitHub Pages |
|---|---|---|---|---|
| **Výchozí doména** | `.pages.dev` | `.vercel.app` | `.netlify.app` | `.github.io/<repo>/` |
| **Přenos dat zdarma** | ♾️ **neomezeně** | 100 GB/měs | 100 GB/měs | ~100 GB (měkký) |
| **Buildy zdarma** | 500/měs | 6 000 min/měs | 300 min/měs | ∞ (Jekyll) |
| **Globální CDN** | ✅ (jedna z nejrychlejších) | ✅ | ✅ | ✅ (Fastly) |
| **Vlastní doména + HTTPS** | zdarma | zdarma | zdarma | zdarma |
| **Deploy z CLI** | `wrangler` | `vercel` | `netlify` | `git push` |
| **Serverless funkce** | ✅ Workers | ✅ | ✅ | ❌ |
| **Háček** | UI techničtější | komerční use jen na placené | podobné Vercelu | jen statika, podcesta `/<repo>/` |

---

## Argumentace pro výběr (do obhajoby)

- **Cloudflare Pages** → jako jediný má **neomezený přenos dat zdarma** a jednu z nejrychlejších sítí. Nejsilnější objektivní argument pro „free hosting".
- **Vercel** → nejlepší developer experience, nasazení jedním příkazem, 100 GB/měs nám bohatě stačí. Co už známe.
- **Netlify** → velmi podobné Vercelu, navíc drag-&-drop deploy a formuláře zdarma.
- **GitHub Pages** → nulová závislost navíc, web žije přímo u kódu, ideální pro školní odevzdání a transparentnost.

**Pro náš statický web je technicky jedno, který zvolíme** — všechny zvládnou jeden HTML + obrázky bez problému. Rozhoduje tedy argumentace, ne technické omezení.

---

## Postup nasazení (až padne rozhodnutí)

1. Připravíme projekt pro daný host (u statiky většinou bez konfigurace).
2. Přihlášení přes účet (`vercel login` / `npx wrangler login` / push do GitHub repo).
3. Deploy → veřejný odkaz, který pošleme učiteli.
