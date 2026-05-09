# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt

Einstellungskosten-Kalkulator — eine clientseitige Single-Page-App (reines HTML/CSS/JS, kein Build-Schritt), gehostet via GitHub Pages. Das Backend läuft als Supabase Edge Function (`ba-proxy`), die als Proxy zur offiziellen BA-Statistik-API dient.

## Entwicklung

Kein Build-Schritt, kein Package-Manager. Die App direkt im Browser öffnen:

```
index.html  →  im Browser öffnen (Doppelklick oder Live Server)
```

Für die Supabase-Integration muss `config.js` lokal vorhanden sein (nie committen — steht im `.gitignore`):

```js
const SUPABASE_URL     = 'https://XXXXXXXXXX.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...';
```

Edge Function deployen (nach Änderungen an `supabase/functions/ba-proxy/`):

```bash
supabase functions deploy ba-proxy
```

## Architektur

**`index.html`** enthält die gesamte App — HTML, CSS (`:root`-Variablen, kein Framework) und JavaScript in einem einzigen `<script>`-Block. Es gibt keinen Bundler und keine externen JS-Abhängigkeiten außer Google Fonts.

**Datenfluss:**
1. Nutzer gibt Berufsbezeichnung ein → `onBerufInput()` debounced auf `fetchBerufe()` (350ms)
2. `fetchBerufe()` ruft `ba-proxy?action=berufe` ab → Autocomplete-Liste
3. `selectBeruf()` → `fetchEngpass()` lädt Engpass-Daten für den Beruf
4. Alle Eingabefelder triggern `calculate()` → aktualisiert die rechte Ergebnisspalte live
5. `loadBAData()` läuft beim Start und holt allgemeine Marktdaten

**Wichtige globale Zustände:**
- `jobBoards[]` — Array der Jobbörsen-Einträge (wird per Index aus Inline-`oninput`-Handlern mutiert)
- `lastCalc` — letztes Kalkulationsergebnis, wird von `exportCSV()`, `updateEmpfehlung()` und `saveToHistory()` genutzt
- `baMedian` — manuell oder automatisch gesetzter Jahres-Median für den Gehalts-Benchmark
- `lastEngpassData` — gecachte BA-Engpass-Antwort für den aktuell ausgewählten Beruf

**Sicherheitsfunktionen (alle in `index.html` oben im Script):**
- `escHtml(str)` — vor jedem `innerHTML`-Einbau von API- oder Nutzerdaten verwenden
- `escCsv(val)` — vor CSV-Wert-Ausgabe verwenden (Formel-Injection)
- `getHistory()` — localStorage-Zugriff mit try/catch, immer statt direktem `JSON.parse` verwenden

## BA-API-Proxy

Die Supabase Edge Function (`ba-proxy`) unterstützt drei `action`-Parameter:

| action | Zweck |
|--------|-------|
| `berufe&q=…` | Berufsbezeichnung-Autocomplete |
| `engpass&berufLabel=…` | Engpass-Daten inkl. Kriterien |
| `markt&region=Deutschland` | Allgemeine Markteckwerte |

Die BA Statistik-API ist seit Dezember 2025 offiziell, aber noch im Aufbau — manche Regionen liefern leere Werte.

## Deployment

Push auf `main` → GitHub Pages aktualisiert automatisch (Root `/`, Branch `main`).
`config.js` darf **niemals** committet werden.
