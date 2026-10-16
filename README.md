# divoceduo-admin – ukázka administrace webu Divoce

Klikací prototyp administrace pro web dua Divoce ([divoceduo.cz](https://moses-php.github.io/divoceduo.cz/)),
aby si Alice a Adéla mohly vyzkoušet, jak by správa obsahu vypadala.

**Ukázka:** https://moses-php.github.io/divoceduo-admin/

## Co umí

- **Přihlášení** – výběr Alice / Adéla, v ukázce projde jakékoli heslo.
- **Přehled** – nejbližší akce s odpočtem, počty, historie změn (kdo co upravil).
- **Akce** – seznam nadcházejících / proběhlých, přidání, úprava (datum, místo, ceny, odkaz na
  vstupenky, stav prodeje, fotka), koncept vs. zveřejněná, hlavní akce na úvodu, duplikace, smazání.
- **Texty** – sekce „O nás“ (tučně přes `**…**`), odkazy na sítě, popis pro vyhledávače s počítadlem.
- **Galerie** – nahrání fotek z telefonu/přetažením (zmenší se na 1200 px), popisky, skrytí,
  přebarvení do palety webu, změna pořadí přetažením nebo šipkami.
- **Náhled** – živý náhled webu (mobil / počítač), na širokém monitoru vedle editoru.
- **Zveřejnit** – souhrn neveřejných změn a „nasazení“ (v ukázce jen naoko).

## Jak to funguje

Jeden statický `index.html` bez buildu, data v `localStorage` prohlížeče (každý návštěvník má svou
kopii, „obnovit“ v horním pruhu vrátí výchozí stav). Ukázkové fotky v `assets/galerie/` jsou výřezy
jediné fotky dua. Náhled skládá web ze šablony v `siteHTML()` – vychází z `index.html` hlavního webu.

## Ostrá verze

Plán převedení do provozu je v PLAN.md (jen lokálně, není v repu). Zvažované varianty:

Prototyp nemá backend. Pro skutečné ukládání by šlo např.:

- **Pages CMS / Decap CMS** – obsah jako soubory v repu `divoceduo.cz`, přihlášení přes GitHub,
  uložení = commit, GitHub Pages web znovu nasadí. Zdarma, holky by potřebovaly GitHub účet.
- **Supabase / Firebase** – databáze + úložiště fotek + přihlášení e-mailem, web si obsah načítá.
  Přihlášení jednodušší, ale je to další služba.

Lokální náhled: `python -m http.server` v této složce a otevřít http://localhost:8000/.
