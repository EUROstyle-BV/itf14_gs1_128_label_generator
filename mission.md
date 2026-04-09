# Mission: ITF-14 / GS1-128 Barcode Label Generator

## Project Overview

Build a **single-file, self-contained HTML/CSS/JS web application** that generates professional shipping/logistics barcode labels in real time. The app renders two side-by-side panels: a form on the left for input, and a live label preview on the right. No server, no build step — just open `index.html` in a browser.

The target output is a faithful reproduction of the **EUROstyle / Vitalstyle outer-carton shipping label** (reference: `omdoos_ZomerRust_Spray_500ml_06022629.pdf`).

---

## Tech Stack

| Concern | Choice |
|---|---|
| Language | Vanilla HTML5 / CSS3 / JavaScript (ES2022+) |
| Barcode rendering | [bwip-js](https://github.com/metafloor/bwip-js) v4.8.0 via CDN (`bwip-js.min.js`) |
| Fonts | Google Fonts CDN — see Typography section |
| Print/export | Browser native `window.print()` + `@media print` CSS |
| Icons | None — keep the UI industrial/utilitarian |

### Why bwip-js (not JsBarcode)?
- JsBarcode does not reliably support FNC1 mid-string in GS1-128 (crashes or silently drops separators).
- bwip-js v4.8.0 with `parsefnc: true` + explicit `^FNC1` markers encodes GS1-128 correctly.
- bwip-js `itf14` bcid includes the bearer frame natively — no CSS hack needed.
- SVG output — scalable, crisp at any DPI.
- CDN: `https://cdn.jsdelivr.net/npm/bwip-js@4.8.0/dist/bwip-js.min.js`
  - Note: filename is `bwip-js.min.js` — not `bwip-js-min.js`.

---

## Functional Requirements

### 1. Input Panel (left frame)

| Field | Label in UI | Type | Notes |
|---|---|---|---|
| Logo | Logo (afbeelding) | `<input type="file" accept="image/*">` | Load via `FileReader`, display centered at top of label |
| EAN-13 | EAN-code | `<input type="text" maxlength="13">` | Exactly 13 digits; displayed on label and used to build GTIN-14s |
| Product name | Productnaam | `<input type="text">` | Large heading on label |
| Own product code | Art.nr. | `<input type="text">` | Internal SKU; shown in data grid |
| Quantity | Aantal | `<input type="number" min="1">` | Units per box; shown in "Inhoud" row and GS1-128 AI `(37)` |
| Box contents description | Inhoud omschrijving | `<input type="text">` | E.g. `500 ml`; combined with Quantity on label as `{qty} x {description}` |
| Batch number | Batch | `<input type="text" maxlength="20">` | Alphanumeric; GS1-128 AI `(10)`; shown in data grid |
| Production date | Productiedatum | `<input type="date">` | Formatted `YYMMDD` for GS1-128 AI `(11)`; NOT shown in data grid, only encoded in barcode |
| Best-before date (THT) | THT datum | `<input type="date">` | Formatted `DD-MM-YYYY` on label as "THT:" if present; if empty, shows "PROD:" with production date; `YYMMDD` in GS1-128 AI `(15)` if present |
| Paper size | Papierformaat | `<select>` | `a4` = label centered on A4 sheet; `a6` = 105×148mm page |

All fields trigger an **immediate live update** of the right-hand label preview via `input` / `change` event listeners — no submit button.

### 2. Label Preview Panel (right frame)

Renders a faithful reproduction of the label (see **Label Structure** below). Updates in real time.

### 3. Print / Export

A **"Label afdrukken"** button triggers `window.print()`. Before printing, a `<style id="dynamic-print">` element is injected with the correct `@page` rule based on the selected paper size:

| Paper size | `@page` rule | Layout |
|---|---|---|
| A4 (default) | `size: A4 portrait; margin: 0` | `#preview-panel` is 210mm × 297mm; label (105×148mm) is centered via flexbox — ensures quiet zones are never clipped |
| A6 | `size: 105mm 148mm; margin: 0` | Label (85mm × 128mm) centered with 1cm margins on 105mm × 148mm page; compact CSS overrides reduce paddings/font sizes to fit all content in 128mm |

After printing the `afterprint` event clears the dynamic style.

### 4. Save / Open

- **Opslaan**: exports all field values (including logo as base64 data-URL) to a `.json` file named `{productnaam}_{batchnummer}.json`. Includes `papersize` field.
- **Openen**: loads a previously saved `.json`, fills all fields, restores logo, restores paper size selection, and re-renders the label. The user only needs to update the batch number for a new batch run.

---

## Barcode Specifications

### ITF-14 (bottom barcode — with bearer bar frame)

- **Position on label:** Bottom barcode, inside a thick black rectangular bearer-bar frame.
- **Purpose:** Identifies the outer shipping carton (case level).
- **Rendering:** bwip-js `bcid: 'itf14'` — bearer frame is built into the symbology. Full-width SVG.
- **GTIN-14 construction:**
  - Packaging indicator = **`1`** (case level; hardcoded for outer carton labels)
  - Take EAN-13 digits 1–12 (strip the EAN-13 check digit)
  - Prepend packaging indicator `1` → 13 digits total
  - Calculate GS1 modulo-10 check digit → append → **14-digit ITF-14 value**
- **Human-readable text** displayed below barcode: `18711731033609`

#### GTIN-14 Construction Example (Vitalstyle ZomerRust 500ml)

```
EAN-13:              8711731033602
Strip EAN check:     871173103360   (12 digits)
Prepend indicator 1: 1871173103360  (13 digits)
Calculate check:     → check digit = 9
ITF-14 value:        18711731033609  ✓ matches reference label
```

#### Modulo-10 Check Digit Algorithm (GS1 standard)

```javascript
function gs1CheckDigit(digits13) {
  // digits13: string of exactly 13 digits
  let sum = 0;
  for (let i = 0; i < 13; i++) {
    const multiplier = (i % 2 === 0) ? 3 : 1;  // positions 0,2,4,... × 3
    sum += parseInt(digits13[i]) * multiplier;
  }
  return (10 - (sum % 10)) % 10;
}
// Verify: gs1CheckDigit("1871173103360") === 9  ✓
// Verify: gs1CheckDigit("0871173103360") === 2  ✓
```

> Note: GS1 standard counts from the RIGHT, but bwip-js ITF14 expects the full 14-digit string. The algorithm above processes left-to-right with even (0-indexed) positions × 3 — verify both test cases above.

---

### GS1-128 (upper barcode — no bearer frame)

- **Position on label:** Upper barcode zone, above the ITF-14 barcode, no thick border.
- **Purpose:** Encodes structured logistics data using Application Identifiers.
- **Rendering:** bwip-js `bcid: 'code128'`, `parsefnc: true`. Full-width SVG.
- **GS1 FNC1:** Use `^FNC1` literal in the data string. bwip-js interprets this as FNC1 when `parsefnc: true`.
- **Critical:** AI `(37)` (quantity) is variable-length. A `^FNC1` separator **must** follow it so scanners know where the field ends. Without it scanners misread the barcode.

#### GS1-128 GTIN-14 for AI (02) — packaging indicator `0`

The GTIN-14 inside AI `(02)` uses packaging indicator **`0`** (unit-level) — different from the ITF-14:

```
EAN-13:              8711731033602
Strip EAN check:     871173103360   (12 digits)
Prepend indicator 0: 0871173103360  (13 digits)
Calculate check:     → check digit = 2
GS1-128 GTIN-14:     08711731033602  ✓ matches "(02) 0 8711731 03360 2"
```

#### GS1-128 Data String Construction (bwip-js format)

| AI | Meaning | Format | Value source |
|---|---|---|---|
| `02` | GTIN-14 of contained trade item (PI=0) | 14 digits (fixed) | `0` + EAN-12 + check |
| `37` | Count of trade items | 1–8 digits (variable) | Quantity field — **followed by `^FNC1`** |
| `15` | Best-before date | 6 digits `YYMMDD` (fixed) | THT date field |
| `11` | Production date | 6 digits `YYMMDD` (fixed) | Production date field (omit if empty) |
| `10` | Batch/lot number | 1–20 alphanumeric (variable, last) | Batch number field |

**Encoded data string** passed to bwip-js:
```
^FNC102{GTIN14_pi0}37{qty}^FNC115{YYMMDD_tht}11{YYMMDD_prod}10{batch}
```

**Human-readable text** shown below GS1-128 barcode (with AI parentheses and spaces):
```
(02) 0 8711731 03360 2 (37) 15 (15) 280331 (11) 280206 (10) 06022629
```

---

## Label Structure

**Exact layout from reference PDF** `omdoos_ZomerRust_Spray_500ml_06022629.pdf`.

All sections separated by full-width horizontal rules (`border-bottom: 1.5px solid #000`).
Print target: **105 mm × 148 mm** (A6 portrait). Label background: pure white `#ffffff`.

```
┌────────────────────────────────────────────────┐
│                                                │
│               [LOGO — centered]                │
│                                                │
├────────────────────────────────────────────────┤
│                                                │
│         ZomerRust spray 500 ml                 │  ← very large, bold
│                                                │
├────────────────────────────────────────────────┤
│                                                │
│   Inhoud:                    15 x 500 ml       │  ← large text, two columns
│                                                │
├───────────────────────┬────────────────────────┤
│  Batch:               │  THT: / PROD:*         │
│                       │                        │
│  06022629             │  31-03-2028             │
│                       │                        │
├───────────────────────┼────────────────────────┤
│  EAN-code:            │  Art.nr.:              │
│                       │                        │
│  8711731033602        │  1103360               │
│                       │                        │
├───────────────────────┴────────────────────────┤
│                                                │
│  ██████████████████████████████████████████    │  ← GS1-128 (no frame)
│                                                │
│  (02) 0 8711731 03360 2 (37) 15 (15) 280331   │
│  (11) 280206 (10) 06022629                     │
│                                                │
├────────────────────────────────────────────────┤
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ ████████████████████████████████████████ │  │  ← ITF-14 (bearer frame built into bwip-js)
│  └──────────────────────────────────────────┘  │
│                                                │
│              18711731033609                    │  ← human-readable centered
│                                                │
└────────────────────────────────────────────────┘

\* If THT date is empty, displays "PROD:" with production date instead.
```

### Zone-by-zone CSS Notes

| Zone | Implementation |
|---|---|
| Logo | `display:flex; justify-content:center; padding:12px 0` — `<img>` max-height 80px |
| Product name | Font ~36–42px bold, left-aligned, `padding: 8px 16px` |
| Inhoud row | CSS grid `1fr 1fr`; "Inhoud:" label ~22px bold left; value ~22px right |
| Batch / THT row | CSS grid `1fr 1fr`; `border-right: 1.5px solid #000` on left cell; label 11px, value 20px bold |
| EAN-code / Art.nr. row | Same as Batch/THT row |
| GS1-128 zone | bwip-js SVG full-width; human-readable `<div>` below in 9px monospace; no outer border |
| ITF-14 zone | bwip-js SVG full-width; bearer frame rendered by bwip-js `itf14` bcid; human-readable centered below |

---

## Visual Design

### Label (right panel — printable)
- **Background:** `#ffffff`
- **All text and borders:** `#000000` only (no grey, no color)
- **Dividers:** `border-bottom: 1.5px solid #000` (horizontal), `border-right: 1.5px solid #000` (vertical in grid)
- **ITF-14 bearer frame:** rendered natively by bwip-js `itf14` bcid
- **Typography:**
  - Product name: `'Archivo Black'`, 900 weight, 36–42px
  - "Inhoud:" + value: `'Archivo Black'`, 700 weight, 22–28px
  - Grid labels (Batch, THT, EAN-code, Art.nr.): `'IBM Plex Mono'`, 11px, normal
  - Grid values: `'IBM Plex Mono'`, 20px, 700 weight
  - Barcode human-readable text: `'IBM Plex Mono'`, 9–11px, normal

### Input Panel (left panel)
- **Background:** `#1a1d23`
- **Text:** `#e8e8e8`
- **Input fields:** dark background `#2a2d35`, border `#3a3d45`, white text; focus glow `#4a9eff`
- **Labels:** `#9ca3af`, 11px, uppercase tracking
- **"Label afdrukken" button:** full-width, prominent, `background: #2563eb`
- **App title:** "📦 Label Generator" at top of left panel

---

## Validation & Error Handling

| Condition | Behaviour |
|---|---|
| EAN-13 ≠ 13 digits | Red border on input; barcodes replaced by `⚠ Ongeldige EAN-13` warning |
| EAN-13 check digit wrong | Orange border on input; show `⚠ Controlecijfer onjuist` — barcodes still render |
| Required field empty | Show `—` placeholder on label; do not crash barcode render |
| Logo > 2 MB | Show inline warning; still load |
| THT date empty | Omit AI `(15)` from GS1-128 string |
| Production date empty | Omit AI `(11)` from GS1-128 string |

---

## Reference Test Data (pre-fill on load)

Pre-fill these values via JavaScript `DOMContentLoaded` so the label renders correctly on first open:

| Field | Value |
|---|---|
| EAN-code | `8711731033602` |
| Productnaam | `ZomerRust spray 500 ml` |
| Art.nr. | `1103360` |
| Aantal | `15` |
| Inhoud omschrijving | `500 ml` |
| Batch | `06022629` |
| Productiedatum | `2028-02-06` |
| THT datum | `2028-03-31` |

**Expected generated barcode values:**
- GS1-128 GTIN-14 (AI 02, PI=0): `08711731033602`
- ITF-14 GTIN-14 (PI=1): `18711731033609`
- Full GS1-128 bwip-js data string: `^FNC10208711731033602371515280331112802061006022629` (with `^FNC1` after AI 37 qty if followed by more fields)

---

## Scanner Compatibility

Tested with **Zebra MC330K** running DataWedge.

Requirements in DataWedge:
- Code 128 decoder: **enabled**
- GS1-128 processing: **enabled**
- Without GS1-128 processing the scanner returns raw data without AI parsing.

The A4-centered print layout ensures the GS1-128 quiet zones (left/right margins) are never clipped by the printer, which was the root cause of scan failures when using `@page { size: A6 }` on an A4 printer.

---

## File Structure

```
itf14-gs1128-generator/
├── index.html                        ← single deliverable (all CSS + JS inline)
├── 404.html                          ← GitHub Pages SPA routing redirect
├── mission.md                        ← this file
├── images/
│   ├── Logo_ECOstyle_black.jpg       ← default logo for /ecostyle
│   ├── Vitalstyle_logo_zw.jpg        ← default logo for /vitalstyle
│   └── Logo_AZstyle_black.jpg        ← default logo for /azstyle
└── reference/
    └── omdoos_ZomerRust_Spray_500ml_06022629.pdf
```

Keep everything in `index.html`. No npm, no build, no server — drag-and-drop portable.

---

## Build History

### Stap 1 — Basisstructuur
Twee-kolom layout, alle invoervelden, live update, printknop.

### Stap 2 — Barcodes met JsBarcode
Eerste implementatie: ITF-14 + GS1-128 via JsBarcode. GTIN-14 check digit algoritme geverifieerd.

### Stap 3 — Migratie naar bwip-js (GS1-128 scanproblemen)
- **Probleem 1:** Scanner las alleen de EAN-code uit de GS1-128. Oorzaak: AI `(37)` is variabele lengte — zonder FNC1-separator weet de scanner niet waar het veld eindigt.
- **Poging:** FNC1 mid-string toevoegen in JsBarcode → JsBarcode crashte hierop.
- **Probleem 2:** GS1-128 verdween helemaal. Oorzaak: de fallback verwijderde alleen de eerste `\xf1`, waardoor de tweede ook fouten gaf.
- **Oplossing:** Migratie naar bwip-js v4.8.0. `bcid: 'code128'` + `parsefnc: true` + `^FNC1` markers → betrouwbare GS1-128 codering. ITF-14 via `bcid: 'itf14'` (bearer frame ingebakken).

### Stap 4 — Printprobleem: label niet leesbaar op A4
- **Oorzaak:** `@page { size: A6 }` plaatste label linksboven op A4-vel. Linker ruszone GS1-128 werd afgesneden door printer. Scanners kunnen zonder ruszone de barcode niet vinden.
- **Oplossing:** `@page { size: A4 portrait }` + preview-panel 210×297mm gecentreerd via flexbox. Label staat nu met ~52mm marge links/rechts — beide barcodes scanbaar. ✓

### Stap 5 — Opslaan en Openen
JSON export/import van alle veldwaarden incl. logo (base64) en papierformaat. Bestandsnaam: `{productnaam}_{batch}.json`.

### Stap 6 — A4 / A6 papierformaat keuze
Dropdown in invoerpaneel. Dynamische `<style id="dynamic-print">` injectie vóór `window.print()`:
- A4: bestaande CSS (label gecentreerd op A4).
- A6: `@page { size: 105mm 148mm }` + compacte CSS-overrides (kleinere paddings, fonts, max-height op barcodes) om alle content in 148mm te laten passen.

### Stap 7 — Brand varianten (ECOstyle, VITALstyle, AZstyle)
- URL-gebaseerde brand detectie: `/ecostyle`, `/vitalstyle`, `/azstyle`
- Elk merk heeft eigen logo-afbeelding in `images/` map en eigen header-tekst
- GitHub Pages SPA routing via `404.html` (spa-github-pages patroon)
- Brand-logo wordt automatisch getoond als default; gebruiker kan overschrijven met eigen upload

### Stap 8 — SPA routing bugfix (brand detectie)
- **Probleem:** `404.html` redirect produceert `/?/vitalstyle` waarbij `spaPath` = `'vitalstyle'` (geen leading slash). Check op `'/vitalstyle'` miste altijd → fallback naar EUROSTYLE.
- **Oplossing:** `getBrandFromURL()` doorzoekt de volledige URL-string (`pathname + search + hash`) met `includes('brandname')`.

### Stap 9 — 404 pagina voor ongeldige URLs
- Onbekende sub-paden (bijv. `/hsdgkjshghdhsdk`) tonen een nette "Pagina niet gevonden" pagina i.p.v. stilletjes de default brand te laden.
- Knop "Ga naar de generator" linkt naar `/ecostyle` als standaard.

---

## Out of Scope

- Backend / server-side code
- Database of externe opslag
- PDF export (browser print-to-PDF is voldoende)
- Multi-label batch printing
- SSCC-18 / pallet barcodes (toekomstige fase)
- Configureerbare packaging indicator (hardcoded: GS1-128 PI=0, ITF-14 PI=1)
