# Testplan

## Statuswaarden

- **PASS:** test uitgevoerd en resultaat voldoet.
- **FAIL:** test uitgevoerd en resultaat voldoet niet.
- **BLOCKED:** test kon niet worden uitgevoerd door omgeving of ontbrekende toegang.
- **NOT TESTED:** nog niet uitgevoerd.

## Routes

### Carton

- `/`
- `/ecostyle/`
- `/vitalstyle/`
- `/azstyle/`

### Pallet

- `/pallet.html`
- `/ecostyle/pallet/`
- `/vitalstyle/pallet/`
- `/azstyle/pallet/`

## Testmatrix

| Test | Automatisch | Handmatig/extern |
|---|---:|---:|
| HTTP-response | PASS/FAIL | - |
| Juiste mode | PASS/FAIL | - |
| Titel en header | PASS/FAIL | - |
| Merklogo | PASS/FAIL | - |
| GS1-128-rendering | PASS/FAIL | - |
| ITF-14-rendering | PASS/FAIL/N.v.t. | - |
| Browserconsole | PASS/FAIL | - |
| Mislukte netwerkrequests | PASS/FAIL | - |
| JSON-export | PASS/FAIL | - |
| JSON-import | PASS/FAIL/BLOCKED | - |
| Print Preview | BLOCKED/NOT TESTED | PASS/FAIL |
| PDF-output | BLOCKED/NOT TESTED | PASS/FAIL |
| Fysieke printer | NOT TESTED | PASS/FAIL |
| Fysieke scanner | NOT TESTED | PASS/FAIL |

## Referentiedata

Carton:

```text
EAN: 8711731033602
Product: ZomerRust spray 500 ml
Art.nr.: 1103360
Aantal: 15
Inhoud: 500 ml
Batch: 06022629
Productiedatum: leeg
THT: leeg
```

Pallet:

```text
EAN: 8711731033602
Product: ZomerRust spray pallet
Count: 3700
PROD: 2601
Batch: 06022629
```

Verwachte barcodewaarden:

```text
GS1-128 GTIN-14: 08711731033602
ITF-14: 18711731033609
```

## Per-route procedure

1. Open de route met een unieke cachebuster.
2. Controleer HTTP-status 200 of de afgesproken 404-forwarding.
3. Controleer mode, titel en header.
4. Controleer het juiste merklogo voor merk-cartonroutes.
5. Controleer het aantal SVG-barcodes.
6. Controleer de human-readable barcodewaarden.
7. Registreer console errors en `requestfailed` events.
8. Test JSON-export en controleer bestandsnaam/blob.
9. Test JSON-import met een export uit dezelfde mode.
10. Controleer printregels en printpaginaformaat.
11. Sla resultaten op in de release-evidence.

## Cartonacceptatie

Verwacht:

- root: default EUROSTYLE/carton;
- merkvarianten: juiste merknaam en logo;
- twee SVG-barcodes;
- GS1-128 en ITF-14 zichtbaar;
- A4 en A6 als keuzemogelijkheid;
- custom logo blijft na upload zichtbaar;
- JSON round-trip behoudt velden en custom logo.

## Palletacceptatie

Verwacht:

- één GS1-128-SVG;
- geen ITF-14-zone;
- 2x2-grid met Content, Count, PROD en Batch;
- mode `pallet` bij pathname en SPA-queryroute;
- `dynamic-print` in de head;
- A4 portrait.

## Browserconsole en netwerk

Gebruik browserautomation of DevTools. Markeer als FAIL bij:

- `pageerror`;
- console error;
- ontbrekende barcode-library;
- logo- of fontrequest die onverwacht faalt;
- ongeldige JSON-import die niet foutief wordt gemeld.

Een ontbrekende favicon is alleen een FAIL wanneer favicon-beschikbaarheid onderdeel van de release-scope is.

## Print en PDF

Automatische browserchecks kunnen de aanwezigheid van de printworkflow controleren. De echte Print Preview en PDF-driver zijn omgevingsafhankelijk. Markeer deze als `BLOCKED` wanneer de testomgeving geen OS-printdialoog of PDF-output kan aansturen.

Handmatig:

- schaal 100%;
- browserheaders/footers uit;
- A4 zonder afsnijding;
- A6 op 105 x 148 mm;
- quiet zones volledig aanwezig;
- geen extra pagina's.

## Fysieke printertest

Print minimaal:

- één carton A4;
- één carton A6;
- één pallet A4.

Controleer papierformaat, schaal, contrast, barcodequiet zones en fysieke positie.

## Fysieke scannertest

Referentie: Zebra MC330K met DataWedge.

- Code 128 enabled;
- GS1-128 processing enabled;
- minimaal drie scans per barcode;
- controleer AI(02), AI(37), AI(11), AI(15) en AI(10);
- controleer ITF-14 exact.

## Huidige teststatus

De productieacceptatie is goedgekeurd. De volgende handmatige acceptaties zijn uitgevoerd en geslaagd:

| Onderdeel | Status | Dekking |
|---|---|---|
| ECOstyle pallet | PASS | Palletmodus, juiste merkheader, GS1-128, productie-URL |
| VITALstyle pallet | PASS | Palletmodus, juiste merkheader, GS1-128, productie-URL |
| AZ STYLE pallet | PASS | Palletmodus, juiste merkheader, GS1-128, productie-URL |
| Cartonlabels | PASS | Productie- en browsercontrole |
| PDF-generatie | PASS | Handmatige Chrome-test |
| Fysieke printer | PASS | Handmatige productietest |
| Fysieke barcode scanner | PASS | Handmatige scannertest |
| Exact WMS | PASS | Gescande gegevens verwerkt |

De end-to-end-keten is bevestigd:

```text
Generator -> PDF -> Printer -> Label -> Scanner -> Exact WMS
```

Automatische browserchecks bevestigden daarnaast routes, logo's, barcode-SVG's, console/networkstatus en JSON-export. Een native file chooser kan in sommige browserautomatiseringsruns niet volledig worden gevuld; de handmatige productieacceptatie compenseert die automatiseringsbeperking.
