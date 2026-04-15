# ITF-14 / GS1-128 Label Generator

Always read `mission.md` first before any coding task.

## Key facts

- Output: single `index.html`, no build tools, no npm
- Barcodes: bwip-js CDN (ITF14 + CODE128 formats)
- GS1-128 top, ITF-14 bottom (bearer frame)
- PI=0 for GS1-128 AI(02), PI=1 for ITF-14
- Default test data: EAN `8711731033602`, batch `06022629`; datumvelden starten leeg
- If THT date is empty, label displays "PROD:" with production date instead of "THT:"
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

- **A4:** `#preview-panel` is 210×297mm flex-centered; label is 105×148mm with `min-height`
- **A6:** same `display: flex` centering approach as A4 — `#preview-panel` 105×148mm, `padding: 5mm`, label `width: 95mm` with **no fixed height** (`min-height: unset`)
- **CRITICAL:** Never set a fixed `height` on `#label` in A6 mode. The label uses `flex-direction: column`; a fixed height causes `flex-shrink` to squish barcode zones. The SVGs inside have `height: auto` and don't shrink — they overflow and visually displace barcodes.
- SVG barcodes always size via `width: 100%; height: auto` — do not add `max-height` overrides

## Live URLs

- ECOstyle: `https://labels.eurostyle.nl/ecostyle`
- VITALstyle: `https://labels.eurostyle.nl/vitalstyle`
- AZ Style: `https://labels.eurostyle.nl/azstyle`

Custom domain: `labels.eurostyle.nl` (CNAME → `eurostyle-bv.github.io`)
`404.html` uses `pathSegmentsToKeep = 0` (no repo-name prefix with custom domain)
