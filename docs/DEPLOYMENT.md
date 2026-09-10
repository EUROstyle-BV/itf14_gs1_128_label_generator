# Deployment

## Publicatiemodel

De productiepublicatie gebruikt GitHub Pages met:

```text
Source: Deploy from a branch
Branch: gh-pages
Folder: / (root)
```

Het custom domain is:

```text
https://labels.eurostyle.nl
```

`CNAME` bevat `labels.eurostyle.nl` en wordt door de deploymentworkflow in het artifact geplaatst.

## Ketenschema

```text
origin/main
  -> GitHub Actions checkout
  -> tijdelijke deploy/
  -> index.html, pallet.html, 404.html, images/ en merkdirectories
  -> JamesIves/github-pages-deploy-action
  -> gh-pages / root
  -> GitHub Pages
  -> labels.eurostyle.nl
```

De workflow schrijft niet rechtstreeks naar een permanente lokale `deploy/`-map. Die map bestaat alleen tijdens de Actions-run.

## Workflow

De actuele workflow staat in `.github/workflows/ci-cd.yml`.

Gebruikte actions:

- `actions/checkout@v4` in de validatie- en deploymentjob;
- `actions/setup-node@v4` met Node.js 24;
- `JamesIves/github-pages-deploy-action@v4.6.1` met `branch: gh-pages`, `folder: deploy` en `clean: true`.

De validatiejob:

1. installeert `html-validate`, ESLint en stylelint-tools;
2. valideert alle HTML-entrypoints;
3. lint aanwezige losse JavaScriptbestanden;
4. blokkeert deployment bij fouten.

De deploymentjob vereist `validate` en gebruikt:

```yaml
permissions:
  contents: write
```

De workflow gebruikt niet het officiële artifactmodel met `configure-pages`, `upload-pages-artifact` en `deploy-pages`.

## Deploymentvalidatie

Controleer vóór en na publicatie:

```powershell
git status --short --branch
git rev-parse origin/main
git rev-parse origin/gh-pages
```

Controleer de deploymentbranch:

```powershell
git show origin/gh-pages:index.html | Select-String -Pattern 'function getModeFromURL|const routeText|window.location.href.includes\(''pallet''\)'
```

Verwacht:

- `function getModeFromURL` aanwezig;
- `const routeText` aanwezig;
- oude `window.location.href.includes('pallet')` afwezig.

Controleer live HTML met een cachebuster:

```powershell
$url = "https://labels.eurostyle.nl/index.html?v=$([DateTimeOffset]::UtcNow.ToUnixTimeMilliseconds())"
$response = Invoke-WebRequest -Uri $url -UseBasicParsing
$response.Content | Select-String 'function getModeFromURL|const routeText|window.location.href.includes'
$response.Headers | Format-List
```

Controleer live browserruntime op een palletroute:

```javascript
({
  url: location.href,
  bodyClass: document.body.className,
  title: document.title,
  mode: getModeFromURL()
})
```

Verwacht op ECOstyle:

```text
bodyClass bevat pallet-mode
title = ECOSTYLE Pallet Label Generator — GS1-128
```

Herhaal dit voor VITALstyle en AZ STYLE.

## Actieve deployment controleren

Controleer GitHub Pages:

```powershell
gh api repos/EUROstyle-BV/itf14_gs1_128_label_generator/pages
```

Verwacht:

```json
{
  "source": {
    "branch": "gh-pages",
    "path": "/"
  }
}
```

Controleer de laatste Pages-build:

```powershell
gh api repos/EUROstyle-BV/itf14_gs1_128_label_generator/pages/builds/latest
```

Controleer deployments:

```powershell
gh api 'repos/EUROstyle-BV/itf14_gs1_128_label_generator/deployments?environment=github-pages&per_page=20'
```

De actieve Pages-build moet verwijzen naar de actuele deploymentinhoud en niet naar een orphan of verouderde commit.

## Rollback

1. Noteer de huidige werkende `origin/gh-pages`-SHA.
2. Stop nieuwe releases.
3. Controleer of de vorige bekende deploymentcommit beschikbaar is.
4. Publiceer de vorige bekende `gh-pages`-inhoud opnieuw.
5. Controleer `index.html`, merklogo's, GS1-128 en ITF-14 live.
6. Controleer de Pages-buildstatus en responseheaders.
7. Registreer oorzaak, impact, SHA's en herstelactie.

Een rollback wordt niet uitgevoerd door losse live-bestanden handmatig te vervangen.

## Bekende mismatchstatus

Tijdens de analyse werd vastgesteld dat `main` en `gh-pages` een nieuwe `getModeFromURL()` bevatten, terwijl de custom domain tijdelijk een oudere `index.html` serveerde. De Pages API verwees toen naar orphan commit `1a65b1f` en de live `Last-Modified`-waarde was ouder dan de actuele deployment. De exacte oorzaak tussen Pages-build, custom domain en cache bleef op dat moment `UNDER INVESTIGATION`.
