# ITF-14 / GS1-128 Barcode Label Generator

A single-file, self-contained HTML/CSS/JavaScript web application for generating professional shipping and logistics barcode labels in real time.

## Overview

This application generates **EUROstyle / Vitalstyle outer-carton shipping labels** with:
- **GS1-128 barcode** (top) — encodes product and batch information
- **ITF-14 barcode** (bottom) — shipping carton identification with bearer frame
- **Live form-to-preview synchronization** — no refresh needed
- **Print and export functionality** — save to JSON or print directly

## Features

✨ **No dependencies, no build tools** — single `index.html` file  
✨ **Real-time preview** — form updates instantly refresh the label  
✨ **Professional barcode rendering** — GS1-compliant barcodes using bwip-js  
✨ **Responsive panels** — input form on left, live preview on right  
✨ **Print-optimized** — A4 and A6 paper size support  
✨ **Save/Load** — export label data as JSON, reload anytime  - **Public hosting ready** — open deployment at `labels.eurostyle.nl` (no auth required)
## Getting Started

1. **Open the application:**
   ```
   index.html
   ```
   Simply open this file in any modern web browser (Chrome, Firefox, Safari, Edge).

2. **Fill in the form** (left panel):
   - Logo (image file)
   - EAN-13 Code
   - Product Name
   - Article Number (Art.nr.)
   - Quantity
   - Box Contents Description
   - Batch Number
   - Production Date
   - Best-Before Date (THT) — displays as "THT:" on label if present; if empty, shows "PROD:" with production date
   - Paper Size (A4 or A6)

3. **View the label** (right panel):
   The label preview updates automatically as you type.

4. **Print or Export:**
   - **Print:** Click "Label afdrukken" to print the label
   - **Save:** Click "Opslaan" to download label data as JSON
   - **Load:** Click "Openen" to load a previously saved label

## Technical Stack

| Component | Technology |
|---|---|
| **Markup** | HTML5 |
| **Styling** | CSS3 |
| **Logic** | Vanilla JavaScript (ES2022+) |
| **Barcodes** | [bwip-js](https://github.com/metafloor/bwip-js) v4.8.0 (CDN) |
| **Fonts** | Google Fonts (Archivo Black, IBM Plex Mono) |

## Barcode Specifications

### GS1-128 (Top Barcode)
- **Purpose:** Encodes product information, batch, and date data
- **AI(02):** GTIN-14 (product case level code)
- **AI(10):** Batch number
- **AI(11):** Production date (YYMMDD)
- **AI(15):** Best-before date (YYMMDD)
- **AI(37):** Quantity of units in box
- **FNC1 separators:** Used to delimit AI codes

### ITF-14 (Bottom Barcode)
- **Purpose:** Identifies the outer shipping carton
- **Format:** 14-digit GTIN (Packaging Indicator + EAN-12 + Check Digit)
- **Packaging Indicator:** `1` (case/carton level)
- **Bearer Frame:** Thick black rectangular frame around barcode (native to ITF-14 symbology)

#### GTIN-14 Construction Example
```
EAN-13:              8711731033602
Strip EAN check:     871173103360   (12 digits)
Prepend indicator 1: 1871173103360  (13 digits)
Calculate check:     → check digit = 9
ITF-14 value:        18711731033609
```

## Default Test Data

Use these values to verify barcode rendering:
- **EAN-13:** `8711731033602`
- **Batch:** `06022629`
- **Best-Before Date:** `2028-03-31`

**Expected ITF-14 output:** `18711731033609`

## File Structure

```
itf14_gs1_128_label_generator/
├── index.html                  # Main application (single file)
├── JsBarcode.all.min.js        # Local barcode library (optional CDN fallback)
├── .gitignore                  # Git ignore configuration
├── README.md                   # This file
├── mission.md                  # Detailed project specification
├── CLAUDE.md                   # Developer notes
└── reference/                  # Reference materials and examples
```

## Browser Compatibility

- Chrome / Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+

## Print Settings

- **A4 Layout:** Label is centered on an A4 page (210mm × 297mm)
- **A6 Layout:** Label (85mm × 128mm) centered with 1cm margins on a 105mm × 148mm page
- **Margins:** Set to 0mm for best results
- **Scale:** 100% (do not scale in print dialog)

## Saving and Loading

### Export (Opslaan)
Downloads all form data and logo as a JSON file:
```json
{
  "ean": "8711731033602",
  "productName": "ZomerRust Spray",
  "articleNr": "123456",
  "quantity": 24,
  "contentDescription": "500 ml",
  "batch": "06022629",
  "productionDate": "2025-03-31",
  "thtDate": "2028-03-31",
  "logo": "data:image/png;base64,...",
  "paperSize": "a4"
}
```

### Import (Openen)
Load a previously saved JSON file to restore all label data and settings.

## API / Customization

The application is designed for ease of use without requiring coding. However, you can customize:

- **Default values:** Modify the initial form values in the JavaScript section
- **Styling:** Adjust CSS variables in the `<style>` tag
- **Barcode libraries:** Change CDN URLs for bwip-js if needed

## Deployment

This application is designed for **open public hosting** on a custom domain like `labels.eurostyle.nl`.

**Design philosophy:**
- No authentication required — the tool adds value through EUROstyle branding
- No server-side data storage — all processing happens in the browser
- Minimal overhead — single HTML file, no dependencies, no build step

See [DEPLOYMENT.md](DEPLOYMENT.md) for setup and hosting recommendations.

## Privacy & Security

- ✅ All processing happens in your browser
- ✅ No data is stored on any server
- ✅ No cookies or persistent tracking
- ✅ Safe for public deployment

## Troubleshooting

| Issue | Solution |
|---|---|
| Barcodes not rendering | Verify internet connection (CDN access). Check browser console for errors. |
| Print looks cut off | Ensure margins are set to 0mm in print dialog. Use correct paper size selection. |
| Logo not appearing | Verify image format is supported (PNG, JPG, SVG). File size should be <5MB. |
| JSON import fails | Ensure JSON is valid (use a JSON validator). Check file was exported from this app. |

## License

Developed for EUROstyle / Vitalstyle logistics operations.

## Support

For issues or questions, refer to `mission.md` for detailed technical specifications or `CLAUDE.md` for developer notes.

---

**Version:** 1.0  
**Last Updated:** March 2026
