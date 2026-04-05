# 7roast – Projekt-Dokumentation & Schnellreferenz
> Erstellt: 05.04.2026 | Letzte Aktualisierung: 05.04.2026

---

## 1. PROJEKT-ÜBERSICHT

**Name:** 7roast – Specialty Coffee Bestellseite  
**Betreiber:** Matt, Jürgen, Roland & Sven  
**Kontakt:** seven.farsi@gmail.com | 551 171 164  
**Live-URL:** https://sevenfarsi-007.github.io/7roast-test/  
**GitHub Repo:** https://github.com/sevenfarsi-007/7roast-test  
**Backend:** Google Apps Script + Google Sheets  

---

## 2. TECHNISCHE SCHLÜSSELDATEN

### GitHub
| Wert | Detail |
|------|--------|
| Repo | sevenfarsi-007/7roast-test |
| Branch | main |
| Hauptdatei | index.html |
| Live-Deployment | GitHub Pages (automatisch bei Push) |

### Google Apps Script
| Wert | Detail |
|------|--------|
| Projekt-ID | 1X1tBQf8kzIl0z9DBaGZIl93DGYWADsIF2zeNvITdX6SysKBA09TYW_fS |
| Deployed URL | https://script.google.com/macros/s/AKfycbw60UseW-T1jRPmaLIax-mz6vPlk0XG77wUCa4u3CuetxIawCWoZr0r5kX1fE-Gp0e7/exec |
| Ausführen als | Ich (Google-Konto) |
| Zugriff | Jeder |
| Aktuelle Version | 132 (05.04.2026) |

### Google Sheets
| Wert | Detail |
|------|--------|
| Spreadsheet-ID | 178wes8g7dpUKO8s2_ZxlLbEldx9u9nvDT25DFKIY80w |
| Bestellungen (ORDERS_SHEET_GID) | 0 |
| Lagerbestand (STORAGE_SHEET_GID) | 564926814 |
| Zugang (ACCESS_SHEET_GID) | 995540224 |
| Preise (PRICING_SHEET_NAME) | Pricing |

### Apps Script API-Endpunkte
| Endpunkt | Funktion |
|----------|----------|
| ?api=stock | Liefert Live-Lagerbestand als JSON {BRA-01: int, …} |
| ?api=access | Liefert Zugangs-Arrays {friendEmails: […], cofounderNames: […]} |
| POST (body: JSON) | Bestellung speichern |

---

## 3. DESIGN-SYSTEM

| Variable | Wert | Verwendung |
|----------|------|------------|
| --primary | #0d1b2a | Navy – Header, Buttons, Titel |
| --secondary | #c9a84c | Gold – Akzente, Preise, Linien |
| --accent | #e8c15a | Hellgold – Hover-States |
| --bg | #faf8f5 | Seitenhintergrund |
| --card | #ffffff | Karten-Hintergrund |
| --text | #2c2c2c | Haupttext |
| --text-light | #6b6b6b | Sekundärtext |
| --border | #e8e0d5 | Rahmenfarbe |
| --radius | 14px | Eckenradius Karten |

**Fonts:**
- Playfair Display (Serif) → Überschriften h1/h2/h3
- Inter (Sans-Serif) → Fließtext, Labels, Buttons

---

## 4. DATEI-STRUKTUR IM REPO

```
7roast-test/
├── index.html              ← Komplette Website (HTML + CSS + JS, ~77KB)
├── image.png               ← Logo (weites Banner-Format, 2214×788px, Gold auf Navy)
├── logo.png                ← Altes Logo (quadratisch, 1024×1024px) – nicht mehr verwendet
├── brazil.jpg              ← Kaffee-Foto: Brasil Cerrado
├── colombia.jpg            ← Kaffee-Foto: Colombia
├── ethiopia.jpg            ← Kaffee-Foto: Ethiopia Limmu
├── panama.jpg              ← Kaffee-Foto: Panama
├── rwanda.jpg              ← Kaffee-Foto: Rwanda
├── nicaragua.jpg           ← Kaffee-Foto: Nicaragua (noch keine Karte dafür)
├── About us.jpg            ← Foto für About-Us-Sektion
├── hero.mp4                ← Header-Video (AKTUELL LEER – echte Datei noch hochladen!)
├── NOTIZEN_7roast.md       ← Diese Datei
├── DOKU_7roast_05042026.md ← Kurzdoku Session 3
└── DOKU_7roast_Session_2026-04-04.md ← Ausführliche Doku Session 3
```

> ⚠️ **hero.mp4** ist derzeit 2 Bytes (Platzhalterdatei). Die echte Datei muss noch hochgeladen werden (max. 25 MB über GitHub Web, größere Dateien per Git LFS).

---

## 5. KAFFEE-PRODUKTE & PRODUKT-IDs

| Produkt | ID | Datei |
|---------|-----|-------|
| Brasil Cerrado | BRA-01 | brazil.jpg |
| Colombia | COL-01 | colombia.jpg |
| Rwanda | RWA-01 | rwanda.jpg |
| Ethiopia Limmu | Eth-01 | ethiopia.jpg |
| Panama | Pan-01 | panama.jpg |

---

## 6. PREISSTRUKTUR

| Tier | 125g | 250g | 500g | 1000g | +Ground |
|------|------|------|------|-------|---------|
| normal | 12 | 22 | 40 | 80 | +2.00 |
| friend | 9 | 18 | 35 | 70 | +1.50 |
| cofounder | 7.50 | 15 | 30 | 60 | +1.00 |

---

## 7. KUNDEN-TIER-ERKENNUNG (Auto-Detect)

Das System erkennt den Tier **automatisch** anhand von Name und E-Mail:

- **Co-Founder** → Name stimmt überein mit Spalte B im "Access"-Sheet (z.B. Jürgen_CO, Matt_CO, Roland_CO, Sven_CO)
- **Friend** → E-Mail stimmt überein mit Spalte A im "Access"-Sheet (15 E-Mail-Adressen)
- **Normal** → alle anderen

**Logik (JS):**
```js
function normStr(s) { return String(s||'').trim().toLowerCase(); }
// cofounder hat Priorität über friend
if (cofounderNames.includes(normStr(name))) → cofounder
else if (friendEmails.includes(normStr(email))) → friend
else → normal
```

**Apps Script Access-Sheet Struktur:**
- Spalte A: friend_emails (lowercase)
- Spalte B: cofounder_names (lowercase, z.B. "jürgen_co")

---

## 8. MEHRSPRACHIGKEIT

Unterstützte Sprachen:
| Kürzel | Sprache | Flag |
|--------|---------|------|
| en | English | 🇬🇧 |
| de | Deutsch | 🇩🇪 |
| ru | Русский | 🇷🇺 |
| ka | ქართული | 🇬🇪 |
| fa | فارسی | 🇮🇷 (RTL) |

Alle Texte sind im `T`-Objekt in index.html definiert. Neue Texte immer in **allen 5 Sprachen** ergänzen.

---

## 9. IMPLEMENTIERTE FEATURES (nach Session)

### Session 1–2 (Grundlage)
- Bestellformular mit Google Apps Script Backend
- Live-Lagerbestand via `?api=stock`
- Mehrsprachigkeit (5 Sprachen)
- no-cors entfernt (CORS-Fix)

### Session 3 (05.04.2026)
- [x] Header: 7roast-Logo zentriert, Sprachauswahl responsiv (Mobile: zentriert / Desktop: rechts)
- [x] About Us: Zieltext + Preisaufschlüsselung (70/15/8/7%) in allen 5 Sprachen
- [x] Preistabelle Mobile: +Ground als separate Zeile unterhalb
- [x] Order-Ansicht: Berechneter Tier-Preis wird im Warenkorb angezeigt
- [x] Platzhalter-Übersetzungen: Name/E-Mail/Notes in allen 5 Sprachen
- [x] Pflichtfelder: Name + E-Mail required
- [x] Logo (image.png) stilvoll in Header + Footer integriert mit Float-Animation
- [x] Animationen: Page-Load (fadeInUp), Röst-Overlay beim Senden, Erfolgs-Konfetti
- [x] Farbschema: Braun → Navy/Gold (passend zum Logo)

### Session 4 (05.04.2026)
- [x] Auto-Detect Kunden-Tier via `?api=access` + JS-Logik
- [x] Kunden-Type-Auswahl ausgeblendet (wird automatisch ermittelt)
- [x] Logo: logo.png → image.png (breiteres Banner-Format, besser lesbar)
- [x] onerror-Fallback für alle Kaffee-Bilder (Gradient-Hintergrund wenn Bild fehlt)
- [x] Apps Script: Version 132 mit neuem `?api=access` Endpunkt deployed
- [x] Fehler behoben: TypeError in onTierChange (Referenz auf entferntes Element)

### Session 5 (05.04.2026)
- [x] Kaffee-Bilder: Alle URLs → lokale Dateien (brazil.jpg, colombia.jpg, etc.)
- [x] About-Us-Foto: Google Sites URL → About us.jpg (lokal)
- [x] Hero-Video: `<video>`-Element mit hero.mp4 eingebunden, opacity: 0.45
- [x] Hero-Overlay bleibt aktiv → Text immer gut lesbar

---

## 10. WORKFLOW: ÄNDERUNGEN VORNEHMEN

### A) HTML/CSS/JS ändern (index.html)
1. Tab öffnen: https://github.com/sevenfarsi-007/7roast-test/edit/main/index.html
2. Aktuellen Stand lesen: https://raw.githubusercontent.com/sevenfarsi-007/7roast-test/main/index.html
3. Änderungen via JavaScript im Editor-Tab bauen (`fetch` raw URL → string replace → `navigator.clipboard.writeText`)
4. Im GitHub-Editor: Klick in den Editorbereich → Cmd+A → Cmd+V
5. "Commit changes" → Commit-Message eingeben → "Commit directly to main"
6. Warten ~30 Sek → Live-Site testen: https://sevenfarsi-007.github.io/7roast-test/

### B) Apps Script ändern
1. Tab öffnen: https://script.google.com/u/0/home/projects/1X1tBQf8kzIl0z9DBaGZIl93DGYWADsIF2zeNvITdX6SysKBA09TYW_fS/edit
2. Code bearbeiten (Monaco Editor)
3. Cmd+S zum Speichern
4. **NEU DEPLOYEN:** Bereitstellen → Bereitstellungen verwalten → Stift-Icon → "Neue Version" → Bereitstellen
5. ⚠️ IMMER dieselbe Deployment-ID behalten! URL darf sich NICHT ändern.
6. Endpunkt testen: [APPS_SCRIPT_URL]?api=stock bzw. ?api=access

### C) Bilder/Videos hochladen
1. GitHub Repo öffnen: https://github.com/sevenfarsi-007/7roast-test
2. "Add file" → "Upload files"
3. Datei in das Drop-Feld ziehen
4. "Commit directly to main" → "Commit changes"
5. Im index.html mit lokalem Pfad referenzieren: `src="dateiname.jpg"`

### D) hero.mp4 hochladen (echte Datei)
- Unter 25 MB: Über GitHub Web hochladen (wie oben)
- Über 25 MB: Terminal/Git LFS nötig:
  ```bash
  git lfs install
  git lfs track "*.mp4"
  git add .gitattributes hero.mp4
  git commit -m "Add hero video"
  git push
  ```

---

## 11. WICHTIGE REGELN & FALLSTRICKE

| Regel | Detail |
|-------|--------|
| no-cors NICHT verwenden | Fetch-Aufrufe ohne `mode: 'no-cors'` |
| Apps Script Rückgabe | Immer `ContentService.MimeType.JSON` setzen |
| Deployment-URL | Darf sich NIE ändern – immer bestehende Version updaten |
| Deploy-Einstellungen | Ausführen als: Ich / Zugriff: Jeder |
| Neue Texte | Immer in ALLEN 5 Sprachen (en, de, ru, ka, fa) ergänzen |
| RTL-Sprache | Persisch (fa) → body bekommt Klasse `rtl` → CSS-Anpassungen nötig |
| Bilder | Lokal im Repo speichern (nicht externe Links) |
| Große Videos | GitHub Web-Upload max. 25 MB |

---

## 12. GOOGLE SHEETS STRUKTUR

### Sheet: Orders (GID 0)
Spalten: Timestamp, CustomerName, CustomerEmail, Tier, Items (JSON), Notes, Language, Total

### Sheet: Storage (GID 564926814)
Spalten: ProductID, Stock (Gramm)
Beispiel: BRA-01 | 3725

### Sheet: Access (GID 995540224)
| Spalte A | Spalte B |
|----------|----------|
| friend_emails | cofounder_names |
| (15 E-Mail-Adressen) | jürgen_co, matt_co, roland_co, sven_co |

### Sheet: Pricing
| Tier | 125g | 250g | 500g | 1000g | Ground |
|------|------|------|------|-------|--------|
| normal | 12 | 22 | 40 | 80 | 2 |
| friend | 9 | 18 | 35 | 70 | 1.5 |
| cofounder | 7.5 | 15 | 30 | 60 | 1 |

---

## 13. BEKANNTE PROBLEME & LÖSUNGEN

| Problem | Ursache | Lösung |
|---------|---------|--------|
| hero.mp4 leer (2 Bytes) | Beim Umbenennen im GitHub Web-Editor wurde Inhalt geleert | Echte Datei erneut hochladen |
| TypeError: Cannot read 'style' | onTierChange referenzierte entferntes Element | Funktion auf showTierDetectedBox() umgeschrieben |
| Bilder nicht anzeigt | GitHub Blob-URL funktioniert nicht für direkte Bildanzeige | Lokale Dateien im Repo verwenden |
| accessData leer beim ersten Tippen | fetchAccessData() ist async | Daten laden innerhalb 1-2 Sek; Detection funktioniert sobald geladen |
| Apps Script 403/CORS | mode: no-cors entfernen | Fetch ohne no-cors, Apps Script gibt JSON zurück |

---

*Letzte Aktualisierung: 05.04.2026 – Session 5*
