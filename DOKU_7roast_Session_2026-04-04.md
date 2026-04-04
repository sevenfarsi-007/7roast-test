# 7roast – Projektdokumentation
## Session: 04.04.2026

---

## Projektübersicht

| Eigenschaft | Wert |
|---|---|
| **Projekt** | 7roast – Specialty Coffee Bestellseite |
| **Live-URL** | https://sevenfarsi-007.github.io/7roast-test/ |
| **GitHub Repo** | https://github.com/sevenfarsi-007/7roast-test |
| **Apps Script** | CoffeeTBS Live (Projekt-ID: 1X1tBQf8kzIl0z9DBaGZIl93DGYWADsIF2zeNvITdX6SysKBA09TYW_fS) |
| **Google Sheet** | Coffee Project TBS (ID: 178wes8g7dpUKO8s2_ZxlLbEldx9u9nvDT25DFKIY80w) |
| **Apps Script URL** | https://script.google.com/macros/s/AKfycbw60UseW-T1jRPmaLIax-mz6vPlk0XG77wUCa4u3CuetxIawCWoZr0r5kX1fE-Gp0e7/exec |
| **Aktive Version** | Version 131 (04.04.2026, 17:51) |

---

## Systemarchitektur

```
GitHub Pages (index.html)
    |
    |-- fetchLiveInventory()  -->  GET ?api=stock
    |                              |
    |                         Apps Script doGet()
    |                              |
    |                         getProducts()
    |                              |
    |                         getSheetByGid(564926814)
    |                              |
    |                         Google Sheet "Lager" (GID 564926814)
    |                              Spalte A = Kaffee-ID (BRA-01, COL-01, ...)
    |                              Spalte E = totalStock (Gramm)
    |
    |-- Submit (POST)          -->  doPost()
                                    |
                                submitOrder()
                                    |
                                E-Mail an Kunde + Admin
                                    |
                                Bestellung in Sheet eintragen
```

---

## Google Sheet Struktur

### Blatt: Lager (GID = 564926814 = STORAGE_SHEET_GID)
| Spalte | Inhalt |
|--------|--------|
| A | Kaffee-ID (z.B. BRA-01) |
| B | Produktname |
| C | Preis per 1000g |
| D | (unbekannt) |
| E | totalStock (Gramm) |
| F-H | (weitere Daten) |
| Zeile 1 | Kopfzeile (wird übersprungen, Daten ab A2) |

### Blatt: Orders (GID = 0 = ORDERS_SHEET_GID)
- Bestellungen werden hier eingetragen
- Spalte J = "Versendet" (Trigger für E-Mail-Versand)

### Apps Script Konstanten (Code.gs, Zeile 1-14)
```javascript
const SPREADSHEET_ID = '178wes8g7dpUKO8s2_ZxlLbEldx9u9nvDT25DFKIY80w';
const ORDERS_SHEET_GID = 0;
const STORAGE_SHEET_GID = 564926814;
const SENDER_EMAIL_ADDRESS = 'seven.roast@gmail.com';
const LOGO_FILE_ID = '189TSLVRxg-cjUJNpfcBuW5cgSzzOujoe';
const PRICING_SHEET_NAME = 'Pricing';
const ACCESS_SHEET_NAME = 'Access';
const CACHE_TTL_SECONDS = 60; // 1 min
```

---

## Apps Script – Funktionsübersicht (Code.gs, 561 Zeilen)

| Funktion | Zeile | Beschreibung |
|----------|-------|--------------|
| `norm_(s)` | 16 | Hilfsfunktion: String normalisieren (lowercase, trim) |
| `getSheetByName_(name)` | 20 | Öffnet Sheet per Name via SPREADSHEET_ID |
| `loadPricing_()` | 24 | Lädt Preistabelle aus "Pricing"-Sheet (gecacht) |
| `doGet(e)` | 128 | GET-Handler: ?api=stock → JSON mit Lagerbeständen |
| `doPost(e)` | 160 | POST-Handler: Bestellung entgegennehmen, submitOrder() aufrufen |
| `getSheetByGid(gid)` | 190 | **NEU (04.04.26)** Sucht Sheet anhand GID-Nummer |
| `getProducts()` | 200 | Lädt alle Produkte aus STORAGE_SHEET_GID (gecacht, 5 min) |
| `submitOrder(formData)` | 266 | Bestellung verarbeiten, in Sheet schreiben, E-Mail senden |
| `onEdit(e)` | 399 | Trigger: Wenn "Versendet" in Spalte J → E-Mail versenden |
| `processSingleDispatch_()` | 411 | Versendet E-Mail nach Bestellung |

---

## doGet – Live-Lagerbestand Endpunkt

**URL:** `[APPS_SCRIPT_URL]?api=stock`

**Funktionsweise:**
```javascript
function doGet(e) {
  // Prüft ob ?api=stock in der URL steht
  if (e && e.parameter && e.parameter.api === 'stock') {
    // CacheService prüfen (products_v1, 5 min TTL)
    // getProducts() aufrufen wenn kein Cache
    // stockData = { "BRA-01": 3725, "COL-01": 4125, ... }
    return ContentService.createTextOutput(JSON.stringify(stockData))
      .setMimeType(ContentService.MimeType.JSON);
  }
  // Fallback: Alte index.html ausliefern (für alte Seite)
  return HtmlService.createHtmlOutputFromFile('index.html')...
}
```

**Beispiel-Antwort (Stand 04.04.2026):**
```json
{"BRA-01":3725,"COL-01":4125,"RWA-01":375,"Eth-01":25,"Pan-01":500}
```

---

## doPost – Bestellverarbeitung

**Erwartet (POST-Body als text/plain, JSON-encoded):**
```json
{
  "action": "submitOrder",
  "customerName": "...",
  "customerEmail": "...",
  "notes": "...",
  "items": [
    { "id": "BRA-01", "name": "Brasil Cerrado (BRA-01)", "quantity": 250, "style": "Espresso", "grindType": "Ganze Bohne" }
  ],
  "language": "en"
}
```

**Antwort (Erfolg):**
```json
{ "status": "success", "message": "..." }
```

**Antwort (Fehler):**
```json
{ "status": "error", "message": "Fehlermeldung" }
```

---

## Website (index.html) – Struktur

### Produkte / Kaffee-IDs
| ID | Name | Status (04.04.26) |
|----|------|-------------------|
| BRA-01 | Brasil Cerrado | 3725g |
| COL-01 | Colombia | 4125g |
| RWA-01 | Rwanda | 375g (fast leer) |
| Eth-01 | Ethiopia Limmu | 25g (letzter Rest) |
| Pan-01 | Panama | 500g |

### Preisstruktur (GEL)
| 125g | 250g | 500g | 1000g | + Gemahlen |
|------|------|------|-------|------------|
| 12 | 22 | 40 | 80 | +2 |

### Sprachen
Englisch (en), Deutsch (de), Russisch (ru), Georgisch (ka), Persisch (fa/RTL)

---

## Bugs & Fixes – Diese Session (04.04.2026)

### Fix 1: no-cors entfernt (Submit-Handler)

**Problem:** Der fetch()-Aufruf beim Formular-Submit verwendete `mode: 'no-cors'`. Das bedeutete:
- Die Antwort des Apps Scripts war eine "opaque response"
- Kunden sahen IMMER "Erfolg", egal ob die Bestellung ankam oder nicht
- Echte Fehler wurden verschluckt

**Lösung (index.html):**
```javascript
// VORHER (fehlerhaft):
fetch(APPS_SCRIPT_URL, {
  method: 'POST',
  mode: 'no-cors',  // ← Problem!
  headers: { 'Content-Type': 'text/plain;charset=UTF-8' },
  body: JSON.stringify(data)
})
.then(() => { /* immer Erfolg */ })

// NACHHER (korrekt):
fetch(APPS_SCRIPT_URL, {
  method: 'POST',
  // kein 'mode' → normaler CORS-Request
  headers: { 'Content-Type': 'text/plain;charset=UTF-8' },
  body: JSON.stringify(data)
})
.then(response => response.json())
.then(result => {
  if (result.status === "success") { /* Erfolg */ }
  else { throw new Error(result.message); }
})
.catch((err) => { /* echter Fehler */ })
```

### Fix 2: fetchLiveInventory() hinzugefügt (index.html)

**Problem:** Lagerbestände waren hart-kodiert im HTML (keine Live-Daten).

**Lösung:**
```javascript
function fetchLiveInventory() {
  fetch(APPS_SCRIPT_URL + '?api=stock')
    .then(response => response.json())
    .then(stockData => {
      const p = document.getElementById('product');
      Array.from(p.options).forEach(opt => {
        if (opt.value && stockData[opt.value] !== undefined) {
          opt.dataset.stock = stockData[opt.value];
        }
      });
      console.log("Live-Lagerbestand geladen!", stockData);
    })
    .catch(err => console.error("Fehler beim Laden der Bestaende:", err));
}
// Am Ende: setLang('en'); fetchLiveInventory();
```

**GitHub Commit:** `2fdc40a` – "Fix: no-cors entfernt + Live-Lagerabgleich (fetchLiveInventory) hinzugefügt"

### Fix 3: getSheetByGid() fehlte im Apps Script (HAUPTFEHLER)

**Problem:** 
- Apps Script doGet() rief getProducts() auf
- getProducts() (Zeile 200+) rief `getSheetByGid(STORAGE_SHEET_GID)` auf
- Funktion `getSheetByGid` war **nirgendwo definiert**
- Fehler: `ReferenceError: getSheetByGid is not defined (Zeile 196, Datei "Code")`
- Die URL antwortete mit HTTP 503

**Diagnose-Weg:**
1. Console-Fehler: `Failed to fetch` → CORS-Problem vermutet
2. Netzwerk-Request: Status 503 → Apps Script-Fehler
3. URL direkt im Browser geöffnet → echter Fehler sichtbar

**Lösung (Code.gs, vor Zeile 200 eingefügt):**
```javascript
function getSheetByGid(gid) {
  const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
  const sheets = ss.getSheets();
  for (let i = 0; i < sheets.length; i++) {
    if (sheets[i].getSheetId() === gid) return sheets[i];
  }
  throw new Error('Sheet mit GID ' + gid + ' nicht gefunden');
}
```

**Danach:** Neues Deployment → **Version 131** (gleiche URL, neuer Code)

---

## Deployment-Konfiguration (Apps Script)

| Einstellung | Wert |
|---|---|
| Deployment-Name | GitHub |
| Ausführen als | Ich (seven.farsi@gmail.com) |
| Zugriffsberechtigte | Jeder |
| Aktive Version | 131 (04.04.2026, 17:51) |
| URL bleibt gleich | Ja (Deployment-ID unveränderlich) |

> **Wichtig:** Nach jeder Code-Änderung im Apps Script muss ein neues Deployment erstellt werden!
> Bereitstellen → Bereitstellungen verwalten → GitHub auswählen → Stift → Neue Version → Bereitstellen

---

## Test-Ergebnisse (04.04.2026)

| Test | Ergebnis |
|------|----------|
| GET ?api=stock direkt | ✅ JSON mit 5 Produkten zurückgegeben |
| fetchLiveInventory() auf Website | ✅ "Live-Lagerbestand geladen!" in Console |
| Dropdown data-stock Werte | ✅ Live-Daten aus Sheet (z.B. BRA-01: 3725g) |
| Warenkorb hinzufügen | ✅ Artikel korrekt in Basket |
| Bestand-Abzug nach Add | ✅ BRA-01 nach 250g Bestellung: 3475g |
| Submit ohne no-cors | ✅ Echter JSON-Response auslesbar |

---

## Offene Punkte / TODO

- [ ] Karten-Anzeige auf der Website (stock-bra, stock-col etc.) zeigt noch statische Werte – fetchLiveInventory() aktualisiert nur das Dropdown, nicht die Karten-Texte
- [ ] Cache-TTL prüfen: getProducts() cached 5 Minuten – bei Bestellung sofort aktualisieren?
- [ ] onEdit-Trigger testen: Wenn "Versendet" in Spalte J → E-Mail-Versand
- [ ] Error-Handling bei doPost: resultMessage.startsWith('Fehler:') prüfen ob alle Fehlerfälle abgedeckt sind

---

## Nützliche Links

- **Live-Site:** https://sevenfarsi-007.github.io/7roast-test/
- **GitHub Repo:** https://github.com/sevenfarsi-007/7roast-test
- **Apps Script Editor:** https://script.google.com/u/0/home/projects/1X1tBQf8kzIl0z9DBaGZIl93DGYWADsIF2zeNvITdX6SysKBA09TYW_fS/edit
- **Stock API Test:** [APPS_SCRIPT_URL]?api=stock
- **Google Sheet:** https://docs.google.com/spreadsheets/d/178wes8g7dpUKO8s2_ZxlLbEldx9u9nvDT25DFKIY80w

---
*Dokumentiert: 04.04.2026 – Claude Sonnet 4.6*
