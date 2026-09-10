# Troubleshooting

## Incident summary

**Status: RESOLVED**

De route `https://labels.eurostyle.nl/ecostyle/pallet` kwam tijdens de analyse uit op een root-`index.html` met de cartonmodus. De live runtime had geen `getModeFromURL()` en bevatte nog de oudere `window.location.href.includes('pallet')`-logica. GitHub Pages publiceerde een verouderde build.

## Impact

- Palletroutes toonden de dooslayout.
- De live titel bleef een Doos Label Generator-titel.
- `document.body.className` bevatte geen `pallet-mode`.
- Nieuwe routinglogica was lokaal en in de repository aanwezig, maar niet zichtbaar op de custom domain.

## Timeline

De relevante Git-historie bevat onder meer:

- `ae1208b`: pallet mode support voor `/pallet` routes;
- `a3012f1`: vereenvoudigde palletdetectie;
- `82b0e38`: SPA-routing voor merk/palletpaden hersteld;
- `b08bace`: palletrouting en mode-detectie opgelost;
- `ee8d8d5`: pallet mode detection verder vereenvoudigd;
- `57ff931`: lege triggercommit om Pages-publicatie te forceren.

De exacte deployment- en Pages-tijdstippen staan in GitHub Actions en Pages API en moeten bij een incident opnieuw worden opgehaald.

## Evidence

Bevestigd:

- `origin/main` bevat `getModeFromURL()`;
- `origin/gh-pages` bevat `getModeFromURL()`;
- de workflow kopieert `main/index.html` naar tijdelijke `deploy/index.html`;
- de workflow publiceert tijdelijke `deploy/` naar `gh-pages`;
- Pages source is `gh-pages` met path `/root`;
- de live HTML bevatte tijdens de analyse geen `getModeFromURL()`;
- de live HTML bevatte de oude `window.location.href.includes('pallet')`-check;
- de live response had een oudere `Last-Modified`-waarde;
- Pages build metadata verwees tijdens de analyse naar orphan commit `1a65b1f`.

Lokaal bevestigd:

- alle HTML-entrypoints zijn met `html-validate` gevalideerd;
- merklogo's, barcode-SVG's, console/networkstatus en JSON-export zijn browsermatig getest;
- queryrouteroutes voor palletmode werken lokaal met `getModeFromURL()`.

Niet definitief bewezen:

- of de oude live inhoud door een stale Pages-build kwam;
- of de custom domain tijdelijk aan een andere Pages-deployment gekoppeld was;
- of custom-domainvalidatie of GitHub/Fastly-cache de oude inhoud vasthield.

## Root cause status

**RESOLVED**

GitHub Pages publiceerde een verouderde build. De site is handmatig unpublished en daarna opnieuw gepubliceerd vanaf `gh-pages` / `(root)`. Na deze herpublicatie was het probleem niet meer reproduceerbaar.

## Recovery procedure

1. Controleer `gh-pages` en `main` op dezelfde verwachte markers.
2. Controleer Pages source: `gh-pages` / `/`.
3. Controleer de laatste Pages-buildcommit.
4. Forceer geen willekeurige broncodewijziging om cache te omzeilen.
5. Als de actieve Pages-build oud is: maak de site handmatig **Unpublish**.
6. Publiceer opnieuw vanaf `gh-pages` / `(root)`.
7. Controleer live met een cachebuster.
8. Controleer alle drie merk/palletcombinaties.
9. Houd de vorige bekende goede `gh-pages`-SHA beschikbaar voor rollback.

## Validation procedure

Controleer live:

```javascript
({
  url: location.href,
  search: location.search,
  pathname: location.pathname,
  bodyClass: document.body.className,
  title: document.title,
  mode: typeof getModeFromURL === 'function' ? getModeFromURL() : 'missing'
})
```

Verwacht voor ECOstyle pallet:

```text
bodyClass bevat pallet-mode
mode = pallet
title bevat ECOSTYLE Pallet Label Generator — GS1-128
```

Herhaal voor VITALstyle en AZ STYLE. Controleer ook de HTML-bron op `getModeFromURL`, `const routeText` en afwezigheid van de oude parser.

## Preventive actions

- Documenteer Pages source expliciet als `gh-pages` / `/`.
- Bewaar deployment-SHA's bij releases.
- Voeg een post-deployment smokecheck toe.
- Vergelijk live `index.html`-markers met `origin/gh-pages:index.html`.
- Voeg browsertests voor pathname-, query- en hashrouteroutes toe.
- Pin GitHub Actions later op volledige commit-SHA's.
- Plan JSON-schema- en automatische printregressietests; fysieke printer-, scanner- en Exact WMS-acceptatie is voor deze release uitgevoerd en geslaagd.
