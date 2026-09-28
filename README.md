# AKAI APC Key 25 — odovzdávací systém s AI hodnotením

Statická webová aplikácia (`index.html`) + jedna Vercel serverless funkcia
(`api/evaluate.js`), ktorá bezpečne volá Anthropic API a vracia AI hodnotenie
odovzdanej práce. API kľúč sa nikdy neposiela do prehliadača — žije len na
serveri ako premenná prostredia. Rovnaká architektúra ako Audio Lab, Launchpad
Mini, OBS Lab a Insta360 Lab.

## Štruktúra projektu

```
.
├── index.html               # celá aplikácia (frontend, 10 cvičení, tutoriály)
├── MIDI_Keys_Ucebnica.pdf   # kompletná ilustrovaná učebnica (12 kapitol)
├── api/
│   └── evaluate.js           # serverless funkcia - volá Anthropic API
├── package.json
├── .gitignore
└── README.md
```

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

- **Model `claude-haiku-4-5-20251001`**, `max_tokens: 600`.
- **Prompt caching** — systémový prompt rozdelený na veľký spoločný blok
  pravidiel hodnotenia (`cache_control:{type:"ephemeral"}`) + malý premenlivý
  blok s kritériami konkrétneho cvičenia.
- **Evidence-first grading** — bez aspoň jedného priloženého súboru a bez
  zmysluplného komentára (min. ~15 znakov) sa práca nedá odovzdať. Checklist
  sám osebe nestačí na vysoké skóre.
- **Uloženie do localStorage a export/import odovzdaní (.json)** s
  automatickou detekciou duplicít v „Učiteľskom prehľade".

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

## 4. Otestovanie

Vyber cvičenie, rozbaľ tutoriál a over, že sa zobrazujú všetky ilustrácie.
Klikni na „Podrobný výklad — Kapitola N v celej učebnici" a over, že sa PDF
otvorí na správnej strane. Priloz súbor, napíš komentár, odošli — appka
zavolá `/api/evaluate` a vráti skutočné AI hodnotenie.

## Aktualizácia obsahu

Stačí nahradiť `index.html` alebo `MIDI_Keys_Ucebnica.pdf` v repozitári. Ak
zmeníš názov PDF súboru, uprav aj odkazy `href` v `index.html` (hlavička +
odkazy „Podrobný výklad" v jednotlivých tutoriáloch).
