# Einstellungskosten-Kalkulator

Prognostizierte Einstellungskosten mit BA-Marktdaten-Integration.

---

## Projektstruktur

```
einstellungskosten/
├── index.html                          ← Webapp (GitHub Pages)
├── config.js                           ← Deine Supabase-Zugangsdaten (NICHT pushen!)
├── .gitignore
├── supabase/
│   └── functions/
│       └── ba-proxy/
│           └── index.ts                ← Edge Function (Supabase)
└── README.md
```

---

## Setup: Schritt für Schritt

### 1. GitHub Repository anlegen

1. Neues Repo auf github.com erstellen (z. B. `einstellungskosten`)
2. Alle Dateien hochladen **außer** `config.js`
3. Unter **Settings → Pages → Source**: `main` Branch, Root-Ordner `/`
4. Deine Webapp ist erreichbar unter: `https://DEIN-NAME.github.io/einstellungskosten`

### 2. `.gitignore` anlegen

Erstelle eine Datei `.gitignore` im Root mit diesem Inhalt:
```
config.js
```
Damit bleibt `config.js` lokal und kommt nie ins öffentliche Repo.

### 3. Supabase Projekt anlegen

1. Kostenlos registrieren auf [supabase.com](https://supabase.com)
2. Neues Projekt erstellen (Region: **Frankfurt** für EU-Compliance)
3. Unter **Project Settings → API** findest du:
   - `Project URL` → das ist deine `SUPABASE_URL`
   - `anon public` Key → das ist dein `SUPABASE_ANON_KEY`

### 4. `config.js` lokal befüllen

```js
const SUPABASE_URL     = 'https://XXXXXXXXXX.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...';
```

Diese Datei nur lokal öffnen — niemals ins Git-Repo pushen.

### 5. Supabase CLI installieren

```bash
npm install -g supabase
supabase login
```

### 6. Edge Function deployen

Im Projektordner:
```bash
supabase init
supabase link --project-ref DEINE_PROJECT_REF
supabase functions deploy ba-proxy
```

Die `PROJECT_REF` findest du in der Supabase-URL: `https://supabase.com/dashboard/project/HIER`

### 7. Testen

Öffne die Webapp lokal (einfach `index.html` im Browser öffnen) und klicke **BA-Daten aktualisieren**. Wenn der grüne Punkt erscheint — fertig.

---

## BA-API: Was wird abgerufen?

| Datensatz | BA-Endpunkt | Zweck im Tool |
|---|---|---|
| Gemeldete Arbeitsstellen | `EckwerteTabelleSTEA` | Marktlage: Wie viele Stellen konkurrieren? |
| Beschäftigung | `EckwerteTabelleBST` | Berufsfeld-Größe |
| Median-Entgelt | Fallback-Tabelle* | Gehalts-Benchmark |

*Die BA Entgelt-API ist ein separater Endpunkt (`/entgeltatlas`). Für eine exaktere Benchmark-Integration kann dieser später eingebunden werden.

Alle Daten sind offiziell und kostenlos verfügbar (seit Dezember 2025).

---

## Weiterentwicklung

- **Supabase DB**: Kalkulationen dauerhaft speichern (statt nur localStorage)
- **Entgelt-API**: Exakten Median nach Berufsklassifikation abrufen
- **Multi-User**: Login über Supabase Auth
- **Export PDF**: Über Browser-Print oder separate Library

---

## Hinweise

- `config.js` niemals ins Repo pushen
- Die inoffizielle BA Jobsuche-API (`rest.arbeitsagentur.de/jobboerse`) ist **nicht** eingebunden, da die BA diese Nutzung rechtlich kritisch sieht
- Die Statistik-API ist seit Dez. 2025 offiziell — aber noch im Aufbau; manche Regionen liefern ggf. leere Werte
