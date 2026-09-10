# Changelog

Dit overzicht bevat aantoonbare functionele, deployment- en documentatiewijzigingen uit de Git-historie. Automatisch aangemaakte `Deploying to gh-pages`-commits zijn alleen opgenomen waar ze de deploymentlijn verklaren.

## 2026-09-10

| Commit | Wijziging | Validatie/deploymentstatus |
|---|---|---|
| `57ff931` | Lege triggercommit voor Pages-publicatie na source-switch. | Gepusht naar `origin/main`; workflowrun gestart/gecontroleerd in de deploymentanalyse. |
| `ee8d8d5` | Vereenvoudigde palletmode-detectie op basis van route-informatie. | GitHub Actions-run succesvol; daaropvolgende Pages-deploymentcommit `ae6a8b9`. |
| `b08bace` | Herstelde palletrouting en mode-detectie. | GitHub Actions-run succesvol; deploymentcommit `1fc34a1`. |
| `9f8156d` | Expliciete logo-bronstatus voor `placeholder`, `brand` en `custom`; HTML-validatieconfiguratie toegevoegd. | Lokale html-validate en browserchecks uitgevoerd; deploymentcommit `7b8cefd`. |
| `5259260` | CI-validatie blokkerend gemaakt en deploymentbestanden verplicht gecontroleerd. | Eerste volledige run vond bestaande HTML-fouten; latere validatie werd met `.htmlvalidate.json` groen. |
| `0e53e93` | Relatieve merklogo-paden gecorrigeerd. | Lokale browservalidatie van drie merkcartonroutes; deploymentcommit `90b7912`. |

## 2026-09-09

| Commit | Wijziging | Validatie/deploymentstatus |
|---|---|---|
| `6cb00f3` | Merk-specifieke standalone palletdirectories hersteld. | Deploymentcommit `dc5fa8b`. |
| `b46255b` | Merk-specifieke palletdirectories verwijderd ten gunste van `404.html`-routing. | Deploymentcommit `d235147`. |
| `82b0e38` | SPA-routing voor merk/palletpaden hersteld. | Deploymentcommit `b3cfe0e`. |
| `ebee634` | URL-constructie in `404.html` gecorrigeerd voor poortgebruik. | Deploymentcommit `42b3ed1`. |
| `2711447` | SPA-routing in `404.html` vereenvoudigd. | Deploymentcommit `6217646`. |
| `a3012f1` | Palletdetectie vereenvoudigd naar URL-detectie. | Onderdeel van de historische route-aanpak. |
| `d0f449e` | Rootrouting en palletdetectie verbeterd. | Onderdeel van de historische route-aanpak. |

## 2026-04-15 en eerder

| Commit | Wijziging |
|---|---|
| `7da299a` | A6-printlayout hersteld voor barcodepositie. |
| `af167bc` | A6-labelbreedte teruggezet naar 85 mm voor ITF-14-breedte. |
| `f77db45` | THT- en productiedatumvelden bij start leeg gemaakt. |
| `1416ae4` | Custom domain `labels.eurostyle.nl` toegevoegd. |
| `6d68bd6` | Merklogo-afbeeldingen toegevoegd en aan brandconfig gekoppeld. |
| `fe3fb1b` | Merkdetectie uitgebreid naar de volledige URL. |
| `4c13db5` | SPA-routing voor GitHub Pages toegevoegd. |
| `d8871e8` | ECOstyle, VITALstyle en AZstyle merkvarianten toegevoegd. |
| `ace1fd7` | Palletlabelgenerator met GS1-128 toegevoegd. |
| `4dbbd35` | THT/PROD-logica en A6-labelmarges toegevoegd. |
| `7349c50` | Eerste CI/CD-workflow toegevoegd. |
| `55901c5` | Initiële ITF-14/GS1-128 labelgenerator toegevoegd. |

## Deploymentincidenten

De geschiedenis bevat meerdere route-, cache- en Pages-triggercommits. Een Pages-deploymentcommit kan een gegenereerd commitobject zijn zonder dezelfde lineaire branchgeschiedenis als `main`. De actuele incidentstatus en bewijs staan in [TROUBLESHOOTING.md](TROUBLESHOOTING.md) en is bewust niet als definitief opgelost geregistreerd zolang live content, Pages-build en `gh-pages`-SHA niet gelijktijdig aantoonbaar gelijk zijn.
