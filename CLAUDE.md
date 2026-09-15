# ITF-14 / GS1-128 Label Generator

Always read `mission.md` first before any coding task.

## Key facts

- Output: single-file apps — `index.html` (carton) and `pallet.html`, no build tools, no npm
- Barcodes: bwip-js CDN (ITF14 + CODE128 formats)
- **Carton labels:** GS1-128 top, ITF-14 bottom (bearer frame); A4/A6 paper size support
- **Pallet labels:** GS1-128 only; A4/A6 paper size support (NEW)
- PI=0 for GS1-128 AI(02), PI=1 for ITF-14
- Default test data: EAN `8711731033602`, batch `06022629`; datumvelden starten leeg
- If THT date is empty, label displays "PROD:" with production date instead of "THT:"
- Paper size dropdown saves preference in JSON exports; old JSON defaults to A4
- Verify: `gs1CheckDigit("1871173103360") === 9`

## Brand variants

- Known brands: `ecostyle`, `vitalstyle`, `azstyle`
- Default (root URL): `eurostyle`
- Logo images live in `images/` folder:
  - `images/Logo_ECOstyle_black.jpg` → ecostyle
  - `images/Vitalstyle_logo_zw.jpg` → vitalstyle
  - `images/Logo_AZstyle_black.jpg` → azstyle
- Brand names in header: `ECOSTYLE`, `VITALstyle`, `AZ STYLE`

## Brand/URL routing

- GitHub Pages serves `index.html` for all paths via `404.html` SPA redirect
- `404.html` converts `/vitalstyle` → `/?/vitalstyle` (query param key is `/`, value is `vitalstyle`)
- **CRITICAL:** `spaPath` value has NO leading slash — always check with `includes('brandname')`, not `includes('/brandname')`
- `getBrandFromURL()` searches the full URL string (`pathname + search + hash`) to catch all formats
- Unknown/invalid sub-paths (not matching any brand) → show `show404()` page, not default brand
- 404 page "Ga naar de generator" button links to `/itf14_gs1_128_label_generator/ecostyle`

## Print layout

**Carton Labels (index.html + brand variants):**
- **A4:** `#preview-panel` is 210×297mm flex-centered; label is 105×148mm with `min-height`
- **A6:** same `display: flex` centering approach as A4 — `#preview-panel` 105×148mm with `padding: 10mm`, label `width: 85mm` with **no fixed height** (`min-height: unset`)

**Pallet Labels (pallet.html + brand variants):**
- **A4:** `#preview-panel` flex-centered; label is 210mm with `padding: 40px`; `@page { size: A4 portrait }`
- **A6:** `#preview-panel` flex-centered; label is 85mm with `padding: 10mm`; `@page { size: 105mm 148mm }`; responsive fonts (h1: 28px, grid-value: 14px, barcode-text: 7px)

**General Print Rules:**
- **CRITICAL:** Never set a fixed `height` on `#label` in A6 mode. The label uses `flex-direction: column`; a fixed height causes `flex-shrink` to squish barcode zones. The SVGs inside have `height: auto` and don't shrink — they overflow and visually displace barcodes.
- **CRITICAL:** Keep `width: 85mm` for A6 label. SVG intrinsic widths must match (ITF-14: ~85mm, GS1-128: wider). Browsers don't scale SVG UP beyond intrinsic width in print.
- SVG barcodes always size via `width: 100%; height: auto` — do not add `max-height` overrides
- Dynamic CSS injection via `#dynamic-print` style element (cleared after print)

## Live URLs

- ECOstyle: `https://labels.eurostyle.nl/ecostyle`
- VITALstyle: `https://labels.eurostyle.nl/vitalstyle`
- AZ Style: `https://labels.eurostyle.nl/azstyle`

Custom domain: `labels.eurostyle.nl` (CNAME → `eurostyle-bv.github.io`)
`404.html` uses `pathSegmentsToKeep = 0` (no repo-name prefix with custom domain)
