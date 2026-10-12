# AKAI APC Key 25 — odovzdávací systém s AI hodnotením

Statická webová aplikácia (`index.html`) + Vercel serverless funkcie v `api/`,
ktoré bezpečne volajú Anthropic API, hodnotenie ukladajú do databázy a poskytujú
učiteľovi tabuľku známok. API kľúč sa nikdy neposiela do prehliadača — žije len na
serveri ako premenná prostredia. Rovnaká architektúra ako Audio Lab, Launchpad
Mini, OBS Lab a Insta360 Lab.

## Štruktúra projektu

```
.
├── index.html               # celá aplikácia (frontend, 10 cvičení, tutoriály)
├── MIDI_Keys_Ucebnica.pdf   # kompletná ilustrovaná učebnica (12 kapitol)
├── api/
│   ├── evaluate.js           # overí odovzdanie, zavolá Anthropic API a výsledok uloží do databázy
│   ├── config.js             # verejne: aktívna trieda a mená na výber pre žiakov
│   ├── teacher.js            # chránené heslom: tabuľka, triedy, ručné známky, mazanie
│   └── _db.js                # pomocná vrstva nad Upstash Redis (nie je samostatná adresa)
├── package.json
├── .gitignore
└── README.md
```

## Ako to funguje (odovzdávanie do databázy)

- **Trieda a žiaci:** učiteľ vloží zoznam mien (max. 40) a tým vytvorí triedu. Žiaci si pri
  odovzdávaní vyberú meno zo zoznamu — nepíšu ho, preto sa mená nerozchádzajú. Tímy
  (cvičenia 5 a 9, 2–4 členovia s rolami) sa tiež skladajú z mien zo zoznamu.
- **Jedno odovzdanie = jedno cvičenie pre jedného žiaka alebo pre celý tím.** Tímové odovzdanie
  sa započíta každému členovi; odovzdať ho stačí jednému.
- **Hodnotenie ukladá server**, nie prehliadač žiaka. Žiak nič nesťahuje ani neposiela a nemá
  kde upraviť známku. Žiacka kópia hodnotenia v prehliadači slúži len na zobrazenie spätnej väzby.
- **Opakované odovzdanie:** uchová sa posledných 10 pokusov, do tabuľky ide najlepší.
- **Ak AI zlyhá**, odovzdanie sa zaznamená ako „čaká na ručné hodnotenie“ (v tabuľke „…“) a
  známku zadáš ručne. Žiak dostane len orientačný odhad.
- **Učiteľ** (záložka *Učiteľský prehľad*, za heslom) vidí tabuľku žiaci × 10 cvičení, v detaile
  komentár žiaka, odkaz na jeho súbory na Google Drive, role v tíme a hodnotenie AI. Môže zadať
  ručnú známku (platí len pre daného žiaka) a exportovať tabuľku do CSV pre Excel.
- **Nový školský rok:** *Nová trieda* → vlož nový zoznam. Stará trieda ostáva uložená aj s
  hodnoteniami (prepínač hore).
- **Viac projektov, jedna databáza:** zoznam tried je spoločný pre všetky aplikácie (Mixpult,
  Launchpad, MIDI Keys, …), ktoré používajú tú istú databázu — zoznam žiakov vložíš raz.
  Odovzdania a ručné známky sú oddelené podľa projektu (konštanta `PROJECT` v `api/_db.js`).
  Zmazanie triedy zmaže jej známky vo všetkých projektoch.
- Screenshoty sa v databáze neukladajú — AI ich vidí pri odovzdaní a súbory majú žiaci na
  zdieľanom Google Drive (odkaz na Google Drive je pri odovzdaní povinný a uloží sa k odovzdaniu).
- Zo žiackej časti zmizla „učiteľská korekcia známky“ — známku mení len učiteľ v tabuľke.

## Čo je nové v tejto verzii

- **Terminológia opravená podľa reálneho hardvéru** — APC Key 25 má 8
  otočných ovládačov (knobov), nie fadery. Cvičenie 6 je teraz „Ovládače —
  hlasitosť, panoráma a MIDI CC" (predtým nesprávne „Fadery"), cvičenie 5 je
  „Clip Launch" (predtým „Session Mode") — presne podľa oficiálneho názvoslovia
  APC Key 25 a Ableton Live.
- **Každé cvičenie má teraz 2–4 skutočné ilustrácie** z ilustrovanej učebnice
  (schéma zapojenia/princípu, krok vo FL Studiu, krok na zariadení, veľká
  infografika) namiesto jedného generického diagramu — spolu 32 obrázkov
  naprieč 10 cvičeniami, optimalizované na ~15–130 KB/kus.
- **Link na celú učebnicu** — dlaždica „Celá učebnica (PDF)" v hlavičke
  appky otvára `MIDI_Keys_Ucebnica.pdf` v novej karte. Každé cvičenie má
  navyše odkaz priamo na konkrétnu kapitolu a stranu (`#page=N`) v učebnici,
  overené proti reálnemu obsahu PDF.
- **Samotná učebnica bola opravená** — pôvodný súbor mal formátovacie chyby
  (farebné boxy a číslované zoznamy sa lámali cez zlom strany, čím vznikali
  takmer prázdne strany), veľké neoptimalizované obrázky (9,2 MB) a
  nesúlad medzi obsahom a nadpisom kapitoly 7. Opravená verzia: 35 strán
  (namiesto 43), 1,8 MB (namiesto 9,2 MB), žiadne rozbité boxy.

## Architektúra a optimalizácia nákladov (rovnaká ako ostatné projekty)

- **Model `claude-haiku-4-5-20251001`**, `max_tokens: 1500`.
- **Prompt caching** — systémový prompt rozdelený na veľký spoločný blok
  pravidiel hodnotenia (`cache_control:{type:"ephemeral"}`) + malý premenlivý
  blok s kritériami konkrétneho cvičenia.
- **Evidence-first grading** — bez aspoň jedného priloženého súboru a bez
  zmysluplného komentára (min. ~15 znakov) sa práca nedá odovzdať. Checklist
  sám osebe nestačí na vysoké skóre.
- **Odovzdania sa ukladajú do databázy (Upstash Redis)** — export/import `.json` odpadol.

## 1. Nahratie na GitHub

```bash
git init
git add .
git commit -m "MIDI Keys (APC Key 25) - opravena ucebnica a ilustracie"
git branch -M main
git remote add origin https://github.com/<tvoj-ucet>/<repo>.git
git push -u origin main
```

## 2. Import projektu do Vercelu

1. vercel.com → Add New → Project.
2. Vyber svoj GitHub repozitár.
3. Framework Preset: **Other**. Build Command a Output Directory nechaj
   prázdne/predvolené.

## 3. Nastavenie API kľúča

1. Settings → Environment Variables:
   - Name: `ANTHROPIC_API_KEY`
   - Value: kľúč z console.anthropic.com/settings/keys (Scope: Default)
   - Environment: Production, Preview aj Development.
2. Deployments → Redeploy (bez "Use existing Build Cache").
3. API kľúč nikdy neukladaj do súborov v repozitári (ani do `.txt` poznámok) — patrí
   len do Environment Variables vo Verceli.

### Databáza (Upstash Redis)

1. Vo Verceli otvor projekt → **Storage** (alebo **Marketplace**) → **Upstash Redis** → vytvor
   databázu (bezplatný plán, región v EÚ) a **Connect Project**. Ak už databázu máš z iného
   projektu (Launchpad, Mixpult), pripoj tú istú — zoznam tried sa potom zdieľa.
2. Vercel sám pridá premenné `KV_REST_API_URL` a `KV_REST_API_TOKEN` (alebo
   `UPSTASH_REDIS_REST_URL` a `UPSTASH_REDIS_REST_TOKEN` — podporované sú obe dvojice).

### Heslo učiteľa

Vo Vercel → Settings → Environment Variables pridaj `TEACHER_PASSWORD` (môže byť rovnaké vo
všetkých projektoch), Production + Preview + Development, potom **Redeploy**.

### Prvé spustenie

Otvor appku → **Učiteľský prehľad** → prihlás sa heslom → **Nová trieda** → názov (napr.
`4.A 2026/27`) a zoznam žiakov, jeden na riadok → **Uložiť**. (Ak triedu už vytvoril iný projekt
na rovnakej databáze, objaví sa tu automaticky.)

## 4. Otestovanie

Vyber cvičenie, rozbaľ tutoriál a over, že sa zobrazujú všetky ilustrácie.
Klikni na „Podrobný výklad — Kapitola N v celej učebnici" a over, že sa PDF
otvorí na správnej strane. Vyber svoje meno, priloz súbor, napíš komentár,
odošli — appka zavolá `/api/evaluate`, vráti skutočné AI hodnotenie a odovzdanie sa objaví
v učiteľskej tabuľke.

## Aktualizácia obsahu

Stačí nahradiť `index.html` alebo `MIDI_Keys_Ucebnica.pdf` v repozitári. Ak
zmeníš názov PDF súboru, uprav aj odkazy `href` v `index.html` (hlavička +
odkazy „Podrobný výklad" v jednotlivých tutoriáloch).
