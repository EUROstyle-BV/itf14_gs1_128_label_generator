# Architectuur

## Doel en vorm

De ITF-14 / GS1-128 Label Generator is een statische browserapplicatie. Er is geen backend, database, buildstap of server-side labelverwerking. HTML, CSS en JavaScript worden als self-contained pagina's gepubliceerd via GitHub Pages.

Alle labeldata blijft in de browser. Barcode-SVG's worden lokaal in de pagina opgebouwd met `bwip-js`.

## Entry points

| Entry point | Functie |
|---|---|
| `index.html` | Gecombineerde cartonapplicatie met carton- en palletmode via URL |
| `pallet.html` | Standalone palletgenerator |
| `404.html` | GitHub Pages fallback en route-forwarding |
| `ecostyle/index.html` | ECOstyle cartonentrypoint |
| `vitalstyle/index.html` | VITALstyle cartonentrypoint |
| `azstyle/index.html` | AZ STYLE cartonentrypoint |
| `ecostyle/pallet/index.html` | ECOstyle standalone palletentrypoint |
| `vitalstyle/pallet/index.html` | VITALstyle standalone palletentrypoint |
| `azstyle/pallet/index.html` | AZ STYLE standalone palletentrypoint |

De merkentrypoints bevatten grotendeels gekopieerde inline HTML/CSS/JavaScript. Dat is functioneel portable, maar creëert onderhoudsschuld.

## Merken

Ondersteunde merken zijn:

- EUROSTYLE als root/default;
- ECOstyle;
- VITALstyle;
- AZ STYLE.

De root-cartonapplicatie gebruikt een `BRAND_CONFIG` met merknaam en logo-asset. De logo-afhandeling gebruikt expliciete `data-logo-source`-statussen:

- `placeholder` voor de transparante initiële afbeelding;
- `brand` voor het geconfigureerde merklogo;
- `custom` voor een geüpload of geïmporteerd logo.

## Routering en mode

`getBrandFromURL()` zoekt merkidentifiers in pathname, querystring en hash om ook GitHub Pages SPA-vormen te ondersteunen.

`getModeFromURL()` in de actuele root-`index.html` bouwt één routewaarde op:

```javascript
const routeText = (
  window.location.pathname +
  window.location.search +
  window.location.hash
).toLowerCase();

return routeText.includes('/pallet') ? 'pallet' : 'carton';
```

De modeklasse `pallet-mode` wordt vóór `applyBrandSettings()` toegevoegd. CSS gebruikt deze klasse om cartonvelden, palletvelden, labelzones en printlayout te wisselen.

`404.html` zet een onbekend GitHub Pages-pad door naar de rootapplicatie in de vorm `/?/<requested-path>`. De route-informatie blijft in de querystring aanwezig.

## Cartonmode

Cartonlabels bevatten:

- merklogo;
- productnaam;
- inhoud en aantal;
- batch, THT/PROD, EAN en artikelnummer;
- GS1-128 bovenaan;
- ITF-14 onderaan met bearer frame.

A4 centreert het 105 x 148 mm-label op een A4-pagina. A6 gebruikt een 85 mm breed label binnen een 105 x 148 mm-pagina. In A6-modus mag het label geen vaste hoogte krijgen, omdat barcodezones anders door flex-shrink worden samengedrukt.

## Palletmode

Palletlabels bevatten:

- productnaam;
- Content/GTIN-14;
- Count;
- PROD in YYMM-weergave;
- Batch;
- één GS1-128-barcode.

De pallet-`PROD`-waarde wordt voor AI(11) naar `YYMM01` geconverteerd. Palletmode toont geen ITF-14-zone.

## Barcodegeneratie

`bwip-js` v4.8.0 wordt via jsDelivr geladen.

Carton:

- GS1-128 via `bcid: 'code128'` en `parsefnc: true`;
- ITF-14 via `bcid: 'itf14'`;
- GS1-128 gebruikt PI=0 in AI(02);
- ITF-14 gebruikt PI=1;
- AI(37) is variabel en krijgt een FNC1-separator als er volgende velden zijn.

Referentiewaarden:

```text
EAN-13: 8711731033602
GS1-128 GTIN-14 PI=0: 08711731033602
ITF-14 GTIN-14 PI=1: 18711731033609
```

## JSON-import en export

De browser maakt bij export een lokaal JSON-bestand met formulierwaarden en, indien aanwezig, een custom logo als data-URL. Bij import worden de velden teruggezet en wordt de labelpreview opnieuw gegenereerd.

De repository bevat meerdere historische JSON-veldnaamvarianten. Er is nog geen formeel versieerbaar schema; dit staat op de V2-backlog.

## Printarchitectuur

De applicatie gebruikt `window.print()`, `@media print` en dynamisch gevulde `<style id="dynamic-print">`-elementen. De printpagina wordt afhankelijk van de mode en papierkeuze ingericht.

De browserautomatisering kon de echte OS-printdialoog en PDF-driver niet volledig uitvoeren. Fysieke printer- en scanneracceptatie blijft daarom een handmatige productieactiviteit.

## Externe dependencies

- `bwip-js@4.8.0` via jsDelivr;
- Google Fonts: Archivo Black en IBM Plex Mono;
- GitHub Actions voor validatie en deployment.

## Bekende technische schuld

- Gekopieerde inline applicaties per merk en mode;
- inline JavaScript zonder zelfstandige module- of unit-testlaag;
- dynamische npm-installatie zonder repository-lockfile;
- meerdere JSON-formaten zonder schemaVersion;
- URL-routing naast fysieke merkdirectories;
- externe runtime-CDN zonder lokale fallback of SRI;
- beperkte automatische dekking van print, PDF en scannerhardware.
