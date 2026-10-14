# Plán: administrace webu Divoce v ostrém provozu

Stav 14. 10. 2026: klikací ukázka (tohle repo, data jen v prohlížeči). Živý web
`Moses-PHP/divoceduo.cz` je ručně psaný `index.html` na GitHub Pages.

## Cíl

Alice a Adéla se z mobilu přihlásí, upraví akce, texty a galerii a tlačítkem
**Zveřejnit** je pošlou na web. Bez placených služeb, bez databáze, kterou by
bylo potřeba udržovat, a web zůstane statický (rychlý, dobře ho najde Google).

## Doporučená architektura: Git jako úložiště + malý Cloudflare Worker

```
holky (mobil) ──► admin = Cloudflare Worker (stránky adminu + API)
                    │  přihlášení, nahrání fotek, Zveřejnit
                    ▼  GitHub API (token jen na jedno repo)
            repo divoceduo.cz: content.json, assets/galerie/*, index.html
                    │  commit do master
                    ▼
            GitHub Pages → web (do ~1 min)
```

- **Obsah** = `content.json` v repu webu (akce, texty, seznam fotek). Fotky jako soubory
  v `assets/galerie/` – admin je před nahráním zmenší (~1600 px, pár set kB).
- **Jedna šablona** `render(content) → index.html` (JS modul). Používá ji náhled v adminu
  i Worker při zveřejnění, takže náhled = skutečný web. Vznikne převodem dnešního `index.html`.
- **Zveřejnit** = Worker vyrenderuje `index.html` a jedním commitem uloží `content.json`
  + `index.html`. Commit nese jméno holky a seznam změn, který admin už teď sestavuje
  („Alice: upravila cenu u akce …“). Historie i možnost vrátit změny jsou tak zadarmo v gitu.
- **Rozpracované změny** se průběžně ukládají v prohlížeči (nic se neztratí). Při zveřejnění
  Worker ověří, že mezitím nezveřejnila ta druhá (`sha` souboru), jinak nabídne načíst novou verzi.
- **Přihlášení**: dva účty se jménem a heslem, heslo uložené jako hash v secretech Workeru,
  HttpOnly cookie na 90 dní (na telefonu se přihlásí jednou). Admin a API běží na stejné
  doméně, takže stačí cookie `SameSite=Strict`. Pokusy o přihlášení jsou omezené.
- **Denní přegenerování** (Cron Trigger Workeru, 3:00): proběhlý koncert sám zmizí z úvodu
  a přesune se mezi proběhlé. Commit se udělá, jen když se výstup změní.
- **Cena**: 0 Kč (Cloudflare Workers free, GitHub Pages). Doména divoceduo.cz se platí už teď.

### Proč ne jiné varianty

| Varianta | Proč ne |
| --- | --- |
| Pages CMS / Decap CMS | Holky by potřebovaly účty na GitHubu (s povinným dvoufázovým ověřením), obecné rozhraní místo našeho. |
| Supabase / Firebase | Další služba s databází; free projekt Supabase se po týdnu nečinnosti uspí; web by obsah načítal až v prohlížeči, nebo by stejně potřeboval sestavování. |
| WordPress apod. | Placený hosting, aktualizace, bezpečnost – zbytečné na jednu stránku. |

## Fáze

### 0. Rozhodnutí (holky + správce webu, tento týden)
- Připomínky k ukázce: chybí něco (kontakt, video, repertoár, odkazy na Spotify/YouTube)?
- Galerie a seznam „Další koncerty“ na živém webu jsou nový prvek designu – schválit.
- Přihlášení heslem (doporučeno) nebo kódem z e-mailu (Cloudflare Access).
- Kdy přepnout: doporučuji **až po koncertě 31. 10.** Stavět se dá hned, ale mimo živý web.

### 1. Šablona a content.json
- Převést `divoceduo.cz/index.html` na `render(content)`; dnešní obsah převést do `content.json`.
- Kontrola: vyrenderovaný web se pixelově shoduje s dnešním (Playwright snímky mobil + desktop).
- Nové sekce (galerie, další koncerty) se vykreslí jen tehdy, když mají obsah.

### 2. Worker (API)
- `POST /api/login`, `POST /api/logout`
- `GET /api/content` – aktuální `content.json` z repa
- `POST /api/photo` – uloží zmenšenou fotku do `assets/galerie/` (sama se na webu neukáže, dokud ji neodkáže `content.json`)
- `POST /api/publish` – render + commit, kontrola souběhu přes `sha`
- `GET /api/history` – poslední commity pro „Poslední změny“ na přehledu
- Cron: denní přegenerování
- Secrets: GitHub token (GitHub App nebo fine-grained token jen na `divoceduo.cz`, právo Contents),
  klíč pro podpis cookie, hashe hesel. Nic z toho do repa.

### 3. Admin
- Z ukázky: místo `localStorage` jako zdroje dat API; `localStorage` zůstane na rozpracované změny.
- Pryč demo pruh a „obnovit ukázku“; skutečné odhlášení.
- Manifest a ikona → „Přidat na plochu“ na telefonu (otevírá se jako aplikace).

### 4. Test nanečisto
- Worker nejdřív zapisuje do testovacího repa (`divoceduo-test` s vlastními Pages).
- Projít s oběma účty na telefonu: nová akce, změna ceny, nahrání fotky z galerie telefonu,
  souběžné úpravy, vyprodáno, zrušení, smazání.

### 5. Spuštění (začátek listopadu)
- Přepnout Worker na repo `divoceduo.cz`, nastavit hesla, krátký návod (jedna stránka se snímky).
- 20 minut s holkami nad telefony, ať si samy zveřejní první změnu.
- Správce webu si zapne upozornění na commity v repu (GitHub Watch), aby viděl, co jde ven.

### 6. Později (podle potřeby)
- Admin na `admin.divoceduo.cz` – vyžaduje převést DNS domény na Cloudflare (nejdřív zjistit majitele domény u Active24).
- Sdílené rozpracované změny mezi oběma holkami (Cloudflare KV), tlačítko „Vrátit poslední zveřejnění“.

## Změna pravidel po spuštění

Dnes jde na živý web jen to, co potvrdí správce webu. Po spuštění budou **obsah** zveřejňovat
holky samy; změny **kódu a designu** (šablona, Worker) dál jen po potvrzení.

## Odhad práce

Fáze 1–4 zhruba dva večery, fáze 5 jedno sezení s holkami.
