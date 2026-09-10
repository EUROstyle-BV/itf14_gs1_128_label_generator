# Operations

## Releaseprocedure

1. Controleer scope en werkboom.
2. Voer HTML-validatie en beschikbare browsertests uit.
3. Controleer barcode-referentiewaarden.
4. Controleer deploymentartifact en CNAME.
5. Commit met beschrijvende boodschap.
6. Push naar `origin/main`.
7. Wacht op groene GitHub Actions-run.
8. Controleer `origin/gh-pages` en Pages buildstatus.
9. Voer live smokecheck uit.
10. Registreer release-evidence en openstaande onzekerheden.

## Pre-deployment checklist

- [ ] `git status --short --branch` gecontroleerd.
- [ ] Geen ongewenste broncodewijzigingen.
- [ ] Alle HTML-entrypoints gevalideerd.
- [ ] Barcode-referenties onveranderd.
- [ ] Logo-assets bereikbaar.
- [ ] JSON-import/export getest.
- [ ] Rollback-SHA bekend.
- [ ] `CNAME` correct.

## Post-deployment checklist

- [ ] Workflow `Validate Code` groen.
- [ ] Deploymentjob groen of expliciet verklaard.
- [ ] `origin/gh-pages` bevat de verwachte SHA.
- [ ] Pages source is `gh-pages` / `/`.
- [ ] Pages build status is `built`.
- [ ] Live `index.html` bevat `getModeFromURL`.
- [ ] Live `index.html` bevat `const routeText`.
- [ ] Oude `window.location.href.includes('pallet')` ontbreekt live.
- [ ] Carton- en palletroutes gecontroleerd.
- [ ] Responseheaders en `Last-Modified` vastgelegd.

## GitHub Actions controleren

```powershell
gh run list --workflow ci-cd.yml --limit 10
gh run view <RUN_ID> --log-failed
```

Controleer dat `validate` geslaagd is en dat `deploy` de verwachte commit gebruikt.

## gh-pages controleren

```powershell
git fetch origin main gh-pages
git rev-parse origin/main
git rev-parse origin/gh-pages
git show origin/gh-pages:index.html | Select-String 'function getModeFromURL|const routeText|window.location.href.includes'
```

## Pages environment en source

```powershell
gh api repos/EUROstyle-BV/itf14_gs1_128_label_generator/pages
gh api repos/EUROstyle-BV/itf14_gs1_128_label_generator/pages/builds/latest
gh api 'repos/EUROstyle-BV/itf14_gs1_128_label_generator/deployments?environment=github-pages&per_page=20'
```

Verwacht:

```text
source.branch = gh-pages
source.path = /
```

## DNS en custom domain

- Controleer `CNAME` op `labels.eurostyle.nl`.
- Controleer DNS CNAME naar `eurostyle-bv.github.io`.
- Controleer HTTPS-certificaatstatus in Pages settings.
- DNS- of certificaatwijzigingen kunnen tijdelijk `In Progress` tonen.

## Cachecontrole

Vergelijk live met een unieke queryparameter:

```powershell
$url = "https://labels.eurostyle.nl/index.html?v=$([DateTimeOffset]::UtcNow.ToUnixTimeMilliseconds())"
$response = Invoke-WebRequest $url -UseBasicParsing
$response.Headers | Format-List
```

Let op `ETag`, `Last-Modified`, `Age`, `X-Cache` en `Cache-Control`. Een oude live `Last-Modified` of oude markerinhoud wijst op een niet-actuele Pages-build of cachelaag, maar bewijst op zichzelf niet welke.

## Rollback

- Stop nieuwe publicaties.
- Leg live HTML, Pages build-SHA en `gh-pages`-SHA vast.
- Publiceer de vorige bekende goede `gh-pages`-inhoud.
- Controleer alle merk- en palletroutes.
- Communiceer de rollback.
- Registreer incident en root-cause-status.

## Incidentregistratie

Registreer minimaal:

- tijdstip;
- live URL;
- workflowrun en commit;
- Pages-buildcommit;
- `main`- en `gh-pages`-SHA;
- responseheaders;
- browserconsole/networkstatus;
- impact op labelproductie;
- rollbackbesluit.

## Dependencycontrole

- Controleer `bwip-js`-versie en CDN-beschikbaarheid.
- Controleer GitHub Actions-versies.
- Plan pinning op volledige commit-SHA's.
- Voeg een lockfile toe voordat npm-validatietools productie-kritisch worden.
- Herhaal scanner- en printtesten na barcode- of printdependencywijzigingen.
