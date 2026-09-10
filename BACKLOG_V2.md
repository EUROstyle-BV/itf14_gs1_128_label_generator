# V2 Backlog - ITF-14 / GS1-128 Label Generator

Dit backlog is gebaseerd op de technische analyse van 10 september 2026. De prioriteit beschrijft het risico voor productie, barcodecorrectheid, deployment of onderhoud.

## Uitgangspunten

- De huidige barcode-uitvoer is een functioneel contract. Wijzigingen aan GS1-128, FNC1, GTIN-14 of ITF-14 worden alleen geaccepteerd met regressietests.
- Bestaande productie-URL's blijven tijdens de migratie werken.
- Bestaande JSON-exportbestanden blijven minimaal tijdens de overgang leesbaar.
- A4- en A6-printlayout worden apart gevalideerd; visuele gelijkheid met de huidige output is vereist tenzij expliciet anders besloten.
- Inspanning is uitgedrukt in werkdagen voor een enkele developer, inclusief implementatie en gerichte validatie maar exclusief wachttijd op gebruikersacceptatie, printers of scanners.

## P0 Kritiek

P0-items eerst oplossen voordat structurele refactoring of nieuwe functionaliteit wordt gestart.

### P0-01 - Herstel logo-assets voor directe merk-URL's

- **Reden:** Merkpagina's gebruiken relatieve paden zoals `images/Logo_ECOstyle_black.jpg`, terwijl de logo's in de rootmap `images/` staan. Vanuit `/ecostyle/` wordt daardoor naar `/ecostyle/images/...` verwezen.
- **Risico:** Een productie-label kan zonder merklogo worden geopend of geprint. Dit raakt alle directe merk-entrypoints.
- **Betrokken bestanden:** `index.html`, `ecostyle/index.html`, `vitalstyle/index.html`, `azstyle/index.html`, `images/Logo_ECOstyle_black.jpg`, `images/Vitalstyle_logo_zw.jpg`, `images/Logo_AZstyle_black.jpg`.
- **Geschatte inspanning:** 0,5-1 dag.
- **Gereed wanneer:** Alle drie carton-URL's en alle drie pallet-URL's laden het juiste logo zonder 404 in de browserconsole.
- **Status:** Afgerond op 10 september 2026 voor de drie directe merk-cartonpagina's.
- **Wijziging:** De merkconfiguraties verwijzen vanuit `ecostyle/`, `vitalstyle/` en `azstyle/` met `../images/` naar de bestaande root-assets.
- **Validatie:** Alle drie absolute doelpaden bestaan; `git diff --check` is schoon; de diff bevat uitsluitend logo-padwijzigingen.
- **Open controle:** Browservalidatie op alle productie-URL's en fysieke logo-weergave moet bij de P0-releasevalidatie worden uitgevoerd.

### P0-02 - Maak JSON-import compatibel tussen carton- en palletvarianten

- **Reden:** De rootimplementatie en standalone palletbestanden gebruiken verschillende veldnamen en verwachten niet hetzelfde `mode`- of datumformaat.
- **Risico:** Een opgeslagen label kan bij openen als de verkeerde mode worden behandeld, velden verliezen of een foutief label produceren.
- **Betrokken bestanden:** `index.html`, `pallet.html`, `ecostyle/pallet/index.html`, `vitalstyle/pallet/index.html`, `azstyle/pallet/index.html`.
- **Geschatte inspanning:** 2-3 dagen.
- **Gereed wanneer:** Oude carton- en pallet-JSON's uit iedere huidige variant correct laden en opnieuw exporteren zonder verlies van barcodevelden.

### P0-03 - Maak CI-validatie blokkerend en valideer alle entrypoints

- **Reden:** De pipeline valideert alleen root-`index.html`; fouten worden bovendien genegeerd via `continue-on-error` en `node -c` valideert geen HTML-bestand als geheel.
- **Risico:** Defecte merk- of palletpagina's kunnen zonder pipelinefout worden gedeployed.
- **Betrokken bestanden:** `.github/workflows/ci-cd.yml`, `.eslintrc.json`, alle `*.html`-entrypoints.
- **Geschatte inspanning:** 1-2 dagen.
- **Gereed wanneer:** HTML, JavaScript en assetverwijzingen voor alle entrypoints worden gecontroleerd en één echte fout de deployment blokkeert.
- **Status:** Afgerond op 10 september 2026.
- **Wijziging:** `ci-cd.yml` valideert alle HTML-bestanden, gebruikt geen `continue-on-error`, verwijdert de onjuiste `node -c index.html`-controle en maakt verplichte deploymentbestanden expliciet.
- **Validatie:** Workflowstructuur, YAML-syntax, verplichte workflowvelden en blokkerende shellcommando's zijn gecontroleerd; de volledige diff staat in de release-evidence.
- **Open controle:** De workflow moet nog door GitHub Actions op Ubuntu worden uitgevoerd om de definitieve tooloutput van `html-validate` en ESLint te bevestigen.

### P0-04 - Leg barcode-regressietests vast

- **Reden:** GS1-check digits, PI=0/PI=1 en FNC1-separators zijn bedrijfskritische output, maar worden momenteel alleen handmatig gecontroleerd.
- **Risico:** Een kleine refactor kan labels produceren die visueel correct lijken maar door scanners verkeerd worden geïnterpreteerd.
- **Betrokken bestanden:** Nieuwe testbestanden, gedeelde barcodecode zodra die wordt geextraheerd, `mission.md` als bron van referentiewaarden.
- **Geschatte inspanning:** 2-4 dagen.
- **Gereed wanneer:** Tests de bekende waarden `08711731033602`, `18711731033609` en de verwachte GS1-128-datastring controleren, inclusief variabele AI(37) met FNC1.

## P1 Hoog

P1-items volgen direct na de P0-stabilisatie en moeten voor de structurele migratie zijn afgedekt.

### P1-01 - Kies en documenteer één routingstrategie

- **Reden:** De documentatie beschrijft SPA-routing via `404.html`, terwijl de huidige `404.html` een statische foutpagina is. Tegelijk bestaan fysieke merkdirectories en root-mode-detectie.
- **Risico:** Verschillende hosting-URL's kunnen verschillende applicaties of foutpagina's tonen. Links kunnen op custom domain en GitHub Pages-subpath verschillend werken.
- **Betrokken bestanden:** `404.html`, `index.html`, `pallet.html`, `DEPLOYMENT.md`, `README.md`, `CLAUDE.md`, merkdirectories.
- **Geschatte inspanning:** 2-3 dagen.
- **Gereed wanneer:** Custom domain, GitHub Pages-subpath, carton, pallet en ongeldige routes zijn vastgelegd in een route-matrix en geautomatiseerd gecontroleerd.

### P1-02 - Centraliseer route- en asset-URL's

- **Reden:** Absolute links zoals `/ecostyle` zijn niet zonder meer correct onder een GitHub Pages-repositoryprefix; relatieve assetpaden verschillen per directoryniveau.
- **Risico:** Navigatie, logo's of fonts werken op de fallback-hosting niet, ondanks dat de custom domain wel werkt.
- **Betrokken bestanden:** `404.html`, alle HTML-entrypoints, `DEPLOYMENT.md`, deploymentworkflow.
- **Geschatte inspanning:** 1-2 dagen.
- **Gereed wanneer:** Alle ondersteunde routes werken op zowel `labels.eurostyle.nl` als de GitHub Pages-URL.

### P1-03 - Voeg domeinvalidatie toe voor alle barcodevelden

- **Reden:** EAN-validatie bestaat, maar aantallen, YYMM, datums, batchtekens en veldlengtes worden niet volledig gecontroleerd.
- **Risico:** Ongeldige of slecht scanbare GS1-data kan toch worden geprint.
- **Betrokken bestanden:** Alle carton- en palletimplementaties; later de gedeelde validatiemodule; `mission.md` voor regels.
- **Geschatte inspanning:** 2-3 dagen.
- **Gereed wanneer:** Ongeldige waarden geven een duidelijke foutmelding, worden niet gecodeerd en kunnen niet per ongeluk als geldig label worden geprint.

### P1-04 - Maak printlayout regressiebestendig

- **Reden:** A4/A6-layout is gevoelig voor vaste hoogtes, SVG-intrinsic widths, lange productnamen en lange batches.
- **Risico:** Barcodes kunnen worden vervormd, quiet zones kunnen verdwijnen of inhoud kan buiten het papier vallen.
- **Betrokken bestanden:** `index.html`, alle merk-cartonbestanden, `mission.md`, nieuwe visuele/e2e-tests.
- **Geschatte inspanning:** 3-5 dagen.
- **Gereed wanneer:** Standaarddata, lege datums, lange tekst, lange batch en ontbrekend logo correct renderen op A4 en A6.

### P1-05 - Centraliseer de GS1-domeinlogica

- **Reden:** `gs1CheckDigit`, `buildGTIN14`, validatie en GS1-128-opbouw zijn op meerdere plaatsen gekopieerd.
- **Risico:** Een bugfix of nieuwe AI wordt slechts gedeeltelijk toegepast, met variant-specifieke barcodefouten als gevolg.
- **Betrokken bestanden:** `index.html`, `pallet.html`, alle merk- en palletbestanden; nieuwe `src/` of `js/`-modules.
- **Geschatte inspanning:** 4-6 dagen.
- **Gereed wanneer:** Er is één bron voor barcodeberekening en alle huidige routes gebruiken dezelfde uitkomst.

### P1-06 - Introduceer een versieerbaar JSON-schema

- **Reden:** Exportvelden zijn inconsistent (`prodDate`/`proddate`, `paperSize`/`papersize`) en hebben geen versie of migratiemodel.
- **Risico:** Toekomstige wijzigingen breken bestaande bestanden; support wordt afhankelijk van handmatige reparatie.
- **Betrokken bestanden:** Alle save/load-implementaties, `README.md`, `mission.md`, nieuwe schema- en migratiebestanden.
- **Geschatte inspanning:** 2-4 dagen.
- **Gereed wanneer:** Nieuwe exports bevatten `schemaVersion`, een vast contract en een migrator voor alle bekende oude formaten.

### P1-07 - Maak de dependency-loading robuuster

- **Reden:** `bwip-js` en Google Fonts worden direct vanaf externe CDN's geladen zonder integriteitscontrole of applicatiefoutstatus.
- **Risico:** CDN-uitval of gewijzigde inhoud verhindert barcodegeneratie of verandert de output.
- **Betrokken bestanden:** Alle HTML-entrypoints, `mission.md`, eventueel nieuwe lokale `vendor/`-assets.
- **Geschatte inspanning:** 1-3 dagen.
- **Gereed wanneer:** CDN-uitval resulteert in een duidelijke foutmelding en de gekozen dependencystrategie is reproduceerbaar gedocumenteerd.

## P2 Middel

P2-items verbeteren onderhoudbaarheid, kwaliteit en performance nadat productiegedrag is gestabiliseerd.

### P2-01 - Verwijder volledige HTML-duplicatie

- **Reden:** Carton- en palletlogica bestaat in acht vrijwel zelfstandige kopieën.
- **Risico:** Iedere wijziging blijft duur en inconsistent; code review kan niet betrouwbaar vaststellen dat varianten gelijk zijn.
- **Betrokken bestanden:** `index.html`, `pallet.html`, alle merkdirectories, nieuwe applicatiemodules en templates.
- **Geschatte inspanning:** 5-8 dagen.
- **Gereed wanneer:** Merk en mode worden configuratie, niet langer afzonderlijke kopieën van bedrijfslogica.

### P2-02 - Splits presentatie, domeinlogica en opslag

- **Reden:** HTML, CSS, barcodeberekening, routing, import/export en printlogica staan in dezelfde bestanden.
- **Risico:** Wijzigingen aan UI of opslag kunnen ongemerkt barcodegedrag beïnvloeden.
- **Betrokken bestanden:** Alle HTML-entrypoints; nieuwe `js/`/`src/`- en `styles/`-bestanden.
- **Geschatte inspanning:** 4-7 dagen.
- **Gereed wanneer:** Pure logica onafhankelijk getest kan worden en UI-code alleen de applicatiestatus synchroniseert.

### P2-03 - Voeg browser- en visuele regressietests toe

- **Reden:** Er zijn geen tests voor routes, barcode-rendering, JSON round-trip, logo's of print-CSS.
- **Risico:** CI kan syntactisch groene code opleveren terwijl de gebruiker een leeg of verkeerd label ziet.
- **Betrokken bestanden:** Nieuwe e2e/visual-testbestanden, `.github/workflows/ci-cd.yml`, alle entrypoints.
- **Geschatte inspanning:** 3-5 dagen.
- **Gereed wanneer:** Alle zes merk/mode-combinaties, standaarddata, foutdata en printvarianten automatisch worden gecontroleerd.

### P2-04 - Verminder live-renderingkosten

- **Reden:** Iedere toetsaanslag genereert opnieuw één of twee SVG-barcodes.
- **Risico:** Onnodige CPU-belasting en haperingen bij lange invoer of oudere hardware.
- **Betrokken bestanden:** Carton- en palletcode; na refactor de rendercomponenten.
- **Geschatte inspanning:** 1-2 dagen.
- **Gereed wanneer:** Tekstupdates direct blijven reageren, barcode-rendering wordt gedebounced en identieke input gebruikt cache waar zinvol.

### P2-05 - Beperk en optimaliseer logo-import

- **Reden:** Logo's worden volledig als data-URL in geheugen en JSON geladen; de waarschuwing controleert grootte maar beperkt de import niet.
- **Risico:** Zeer grote JSON-bestanden, trage import/export en geheugendruk.
- **Betrokken bestanden:** Cartonbestanden, save/load-logica, eventueel nieuwe image utility.
- **Geschatte inspanning:** 1-2 dagen.
- **Gereed wanneer:** Bestandsgrootte, MIME-type en afbeeldingsafmetingen worden gecontroleerd en grote afbeeldingen worden beheerst verkleind of geweigerd.

### P2-06 - Versterk security-baseline

- **Reden:** Er is geen CSP-strategie, geen SRI voor CDN-scripts en externe `_blank`-links missen `noopener`.
- **Risico:** Grotere impact bij dependency-compromis, ongewenste externe scriptuitvoering of gewijzigde CDN-inhoud.
- **Betrokken bestanden:** Alle HTML-entrypoints, `DEPLOYMENT.md`, eventueel hostingconfiguratie.
- **Geschatte inspanning:** 2-3 dagen.
- **Gereed wanneer:** Externe bronnen zijn beperkt en gedocumenteerd, scriptintegriteit is controleerbaar en links zijn veilig geopend.

### P2-07 - Werk documentatie en operationeel beheer bij

- **Reden:** README, mission, CLAUDE en deploymentdocumentatie beschrijven deels verschillende structuren en routingmodellen.
- **Risico:** Nieuwe developers kunnen een verkeerde entrypoint aanpassen of een deployment breken.
- **Betrokken bestanden:** `README.md`, `mission.md`, `CLAUDE.md`, `DEPLOYMENT.md`, `README_MODES.md`.
- **Geschatte inspanning:** 2-3 dagen.
- **Gereed wanneer:** Architectuur, routes, JSON-schema, testprocedure, releaseprocedure en rollbackprocedure één consistent verhaal vormen.

## P3 Laag

P3-items zijn nuttig, maar blokkeren productieherstel en de eerste refactor niet.

### P3-01 - Verwijder ongebruikte analytics- en debugcode

- **Reden:** `logPageView()` is niet actief en rootbestanden bevatten uitgebreide productielogs.
- **Risico:** Code- en console-ruis; toekomstige developers kunnen denken dat analytics actief is.
- **Betrokken bestanden:** `index.html`, `ecostyle/index.html`, `vitalstyle/index.html`, `azstyle/index.html`.
- **Geschatte inspanning:** 0,5-1 dag.

### P3-02 - Maak dependencyversies reproduceerbaar

- **Reden:** CI installeert tools zonder projectmanifest en runtime-assets worden alleen via CDN geselecteerd.
- **Risico:** Een volgende CI-run kan andere toolversies gebruiken en ander gedrag rapporteren.
- **Betrokken bestanden:** `.github/workflows/ci-cd.yml`, nieuw `package.json`/lockfile of expliciete pinned toolversies.
- **Geschatte inspanning:** 1-2 dagen.

### P3-03 - Voeg toegankelijkheids- en foutstatusverbeteringen toe

- **Reden:** Foutmeldingen zijn niet overal gekoppeld aan velden en de interface heeft beperkte semantische statuscommunicatie.
- **Risico:** Gebruikers missen invaliditeit of printen een onverwacht label, vooral bij toetsenbord- of screenreadergebruik.
- **Betrokken bestanden:** Alle UI-entrypoints; na refactor gedeelde form- en statuscomponenten.
- **Geschatte inspanning:** 1-3 dagen.

### P3-04 - Optimaliseer fonts en externe resources

- **Reden:** Fonts worden extern geladen en kunnen tijdens preview/print layoutverschuivingen veroorzaken.
- **Risico:** Kleine visuele verschillen tussen browsers en onvoorspelbare printmaten bij netwerkproblemen.
- **Betrokken bestanden:** Alle HTML-entrypoints, print-CSS, `DEPLOYMENT.md`.
- **Geschatte inspanning:** 1-2 dagen.

## Veilige implementatievolgorde voor een enkele developer

De volgorde voorkomt dat structurele wijzigingen tegelijk met productiefixes en barcodewijzigingen worden uitgevoerd.

### Stap 1 - Baseline vastleggen

1. Maak een route-matrix voor de zes merk/mode-combinaties, root en ongeldige routes.
2. Leg de huidige barcodewaarden vast voor de referentiedata uit `mission.md`.
3. Leg A4- en A6-screenshots vast met standaarddata, lege datums en lange tekst.
4. Noteer welke huidige JSON-formaten in omloop zijn.

**Stopcriterium:** Er is een reproduceerbare referentie voor functionaliteit, barcode-output en printlayout.

### Stap 2 - P0-productierisico's oplossen

1. Voer `P0-01` uit: herstel assetpaden.
2. Voer `P0-02` uit: maak JSON-loaders compatibel.
3. Voer `P0-03` uit: maak CI volledig en blokkerend.
4. Voer `P0-04` uit: leg barcode-regressietests vast.

Na iedere wijziging: valideer alle routes en controleer de referentiebarcodewaarden.

**Stopcriterium:** De huidige productievarianten leveren dezelfde correcte output en CI kan regressies tegenhouden.

### Stap 3 - Routing en domeinvalidatie stabiliseren

1. Beslis over fysieke routes versus SPA-routing (`P1-01`).
2. Centraliseer base paths en asset-URL's (`P1-02`).
3. Voeg invoer- en domeinvalidatie toe (`P1-03`).
4. Test ongeldige routes, GitHub Pages-subpath en custom domain apart.

**Stopcriterium:** Elke ondersteunde URL opent voorspelbaar de juiste mode en ongeldige data kan niet als geldig label worden afgedrukt.

### Stap 4 - Printcontract beschermen

1. Voeg printgevallen voor A4, A6, lange tekst en lege velden toe (`P1-04`).
2. Los uitsluitend layoutproblemen op die door die tests worden aangetoond.
3. Controleer quiet zones en SVG-breedtes na iedere wijziging.

**Stopcriterium:** De baseline blijft visueel en functioneel gelijk voor ondersteunde invoer.

### Stap 5 - Gedeelde kern extraheren

1. Extraheer eerst GS1-logica (`P1-05`), zonder de UI te veranderen.
2. Laat oude entrypoints tijdelijk dezelfde gedeelde module gebruiken.
3. Voeg het versieerbare JSON-schema en migraties toe (`P1-06`).
4. Maak CDN-loading foutbestendig (`P1-07`).

**Stopcriterium:** Er is één barcodebron, oude JSON's blijven laden en de barcode-output is gelijk aan de baseline.

### Stap 6 - Applicatiestructuur refactoren

1. Verwijder HTML-duplicatie (`P2-01`).
2. Splits domeinlogica, opslag en presentatie (`P2-02`).
3. Laat merk en mode uitsluitend configuratie zijn.
4. Houd bestaande URL's als compatibility entrypoints in stand.

**Stopcriterium:** Een wijziging aan barcode- of opslaglogica hoeft nog maar op één plek te gebeuren.

### Stap 7 - Geautomatiseerde browser- en kwaliteitscontrole

1. Voeg browser- en visuele tests toe (`P2-03`).
2. Voeg debounce/cache toe nadat correctheid is bewezen (`P2-04`).
3. Beperk logo-import (`P2-05`).
4. Versterk security (`P2-06`).

**Stopcriterium:** CI controleert route, barcode, import/export, logo en printgedrag automatisch.

### Stap 8 - Documentatie en opruimen

1. Werk alle projectdocumentatie bij (`P2-07`).
2. Verwijder ongebruikte logs en analytics (`P3-01`).
3. Maak toolingversies reproduceerbaar (`P3-02`).
4. Werk toegankelijkheid en fonts af (`P3-03`, `P3-04`).

**Stopcriterium:** Een nieuwe developer kan lokaal testen, veilig deployen en rollbacken zonder mondelinge kennis.

## Totale inspanningsinschatting

De backlog bevat bewust overlap tussen refactor, tests en validatie. Een realistische planning voor één developer is:

- **P0:** 5-10 werkdagen
- **P1:** 15-25 werkdagen
- **P2:** 18-30 werkdagen
- **P3:** 3,5-8 werkdagen
- **Totaal inclusief volgordelijke regressie- en acceptatietijd:** ongeveer 8-12 weken

De eerste productieverbeteringen zijn binnen de eerste 1-2 weken haalbaar. De volledige v2-refactor moet pas als voltooid worden beschouwd wanneer de route-, barcode-, JSON- en print-baselines automatisch worden gecontroleerd.

## Productiegebruik binnen EUROstyle

Deze backlog is bedoeld voor een applicatie die logistieke labels produceert voor carton- en palletprocessen. Een release is daarom pas gereed wanneer niet alleen de webpagina werkt, maar ook de fysieke barcode op papier scanbaar is.

### Rollen

- **Developer:** implementatie, tests, releasevoorbereiding en technische rollback.
- **Reviewer:** controleert code, testresultaten, barcodewaarden en releasechecklist.
- **Logistics key user:** controleert labelinhoud, scanneruitvoer en werkproces.
- **Release owner:** geeft het formele Go/No-Go-besluit en communiceert productie-impact.

### Productie-invariant

De volgende eigenschappen mogen tijdens de migratie niet ongemerkt veranderen:

- GS1-128 gebruikt AI(02) met GTIN-14 en AI(37) met correcte FNC1-afbakening.
- GS1-128 gebruikt PI=0 en ITF-14 gebruikt PI=1.
- De referentie-EAN `8711731033602` geeft GS1-128 `08711731033602` en ITF-14 `18711731033609`.
- Lege productie- en THT-datums worden niet gecodeerd.
- A4 blijft gecentreerd met behouden quiet zones.
- A6 gebruikt een labelbreedte van 85 mm zonder vaste labelhoogte.
- Bestaande productie-URL's en bestaande JSON-bestanden blijven bruikbaar.

## 1. Rollbackstrategie

### Technische rollback

1. Elke productie-release krijgt een unieke git-tag, bijvoorbeeld `v2.0.0`.
2. De vorige productieversie wordt vastgelegd als `known-good` commit voordat deployment start.
3. Rollback gebeurt door de deployment opnieuw uit te voeren vanaf de vorige known-good commit of door de vorige `gh-pages` artifactversie opnieuw te publiceren.
4. `CNAME`, logo-assets en routebestanden worden bij rollback samen met de applicatieversie hersteld.
5. Een rollback wordt niet uitgevoerd door handmatig losse HTML-bestanden op de server te vervangen.
6. Na rollback worden minimaal de productievalidatie voor één carton- en één palletlabel uitgevoerd.

### Data-rollback

- JSON-bestanden worden lokaal bij de gebruiker opgeslagen; er is geen serverdatabase die moet worden teruggezet.
- Een nieuwe loader moet oude JSON-formaten blijven lezen.
- Een nieuwe exportversie mag oude productiegegevens niet overschrijven voordat de gebruiker een geldig label heeft gevalideerd.
- Bij schemafouten blijft het originele bestand onaangeraakt en wordt een duidelijke foutmelding getoond.

### Besluitregels

- Barcode onscanbaar, verkeerd AI-resultaat of verkeerd GTIN: **direct rollback**.
- Verkeerde merklogo's, verkeerde route of ontbrekende barcode: **direct rollback**.
- Alleen cosmetische afwijking zonder impact op scanbaarheid: release blokkeren voor onderzoek; rollback als herstel niet binnen het releasevenster lukt.
- Documentatiefout zonder gebruikersimpact: release mag worden aangehouden met een opvolgitem, maar niet worden genegeerd.

## 2. Releaseprocedure

### Voorbereiding

1. Selecteer de backlogitems en noteer de release-scope.
2. Controleer dat alle P0- en relevante P1-tests groen zijn.
3. Maak een releasebranch of releasecommit vanaf `main`.
4. Leg de known-good productiecommit vast.
5. Controleer gewijzigde routes, assets, barcodefuncties en JSON-migraties.
6. Maak een changelog met gebruikersimpact, migratie-impact en rollbackcommit.

### Testomgeving

1. Deploy de release eerst naar een tijdelijke of niet-prominente testlocatie.
2. Test alle merken: ECOstyle, VITALstyle en AZ STYLE.
3. Test beide modes: carton en pallet.
4. Voer de scanner- en printerchecklists uit.
5. Test zowel custom domain-gedrag als GitHub Pages-fallback wanneer die in scope is.
6. Laat een reviewer en logistics key user de resultaten aftekenen.

### Productiedeployment

1. Maak de release-tag.
2. Controleer dat CI groen is en deploymentfouten blokkeert.
3. Publiceer naar GitHub Pages volgens de bestaande workflow.
4. Controleer dat `CNAME` en alle verwachte assets in het deploymentartifact zitten.
5. Wacht op propagatie en cacheverversing binnen het afgesproken releasevenster.
6. Voer de productievalidatiechecklist uit.
7. Communiceer Go, beperkte release of No-Go naar betrokken gebruikers.

### Releasevenster

- Releases worden bij voorkeur uitgevoerd buiten piekuren voor labelproductie.
- Tijdens het venster wordt geen tweede functionele wijziging parallel gepubliceerd.
- Er is altijd een verantwoordelijke developer beschikbaar voor rollback.
- De release owner bewaart testresultaten, tag, commit en rollbackbesluit.

## 3. Acceptatiechecklist scanners

Gebruik minimaal de Zebra MC330K met DataWedge, omdat deze combinatie in de mission als referentiescanner is vastgelegd.

### Scannerconfiguratie

- [ ] Code 128-decoder is enabled.
- [ ] GS1-128 processing is enabled.
- [ ] De scanner is verbonden met het afgesproken testdevice.
- [ ] De testomgeving gebruikt dezelfde DataWedge-profielinstellingen als productie.
- [ ] Scanneroutput wordt ongewijzigd vastgelegd voor vergelijking.

### Cartonlabel

- [ ] Scan GS1-128 met alleen AI(02) en controleer de PI=0-GTIN.
- [ ] Scan GS1-128 met AI(37) en controleer dat het aantal als één veld wordt gelezen.
- [ ] Controleer de FNC1-grens na variabele AI(37).
- [ ] Scan met THT, productiedatum en batch.
- [ ] Scan zonder THT en controleer dat AI(15) ontbreekt.
- [ ] Scan zonder productiedatum en controleer dat AI(11) ontbreekt.
- [ ] Scan ITF-14 en controleer exact `18711731033609` voor de referentie-EAN.
- [ ] Controleer dat scanneroutput niet alleen de EAN toont en geen velden samenvoegt.

### Palletlabel

- [ ] Scan AI(02) met PI=0.
- [ ] Scan count via AI(37) en controleer de juiste waarde.
- [ ] Scan `PROD` als AI(11) met verwachte `YYMM01`-conversie.
- [ ] Scan batch als laatste variabele veld.
- [ ] Controleer dat de palletbarcode geen ITF-14 bevat.

### Scanner Go/No-Go

- [ ] Alle verplichte velden worden correct geparsed.
- [ ] Geen onverwachte truncatie, concatenatie of extra tekens.
- [ ] Minimaal drie scans per testlabel slagen.
- [ ] Een mislukt verplicht testscenario is No-Go voor productie.

## 4. Acceptatiechecklist printers

### Browser en printdialoog

- [ ] Test in de productioneel ondersteunde browser, minimaal Chrome of Edge.
- [ ] Printscale staat op 100%.
- [ ] Printermarges staan op 0 mm waar de printer dit ondersteunt.
- [ ] Headers en footers van de browser zijn uitgeschakeld.
- [ ] De juiste papieroriëntatie is geselecteerd.
- [ ] De printpreview toont geen extra pagina's.

### A4-carton

- [ ] `@page` is A4 portrait.
- [ ] Het label staat gecentreerd op het vel.
- [ ] GS1-128 quiet zones zijn links en rechts volledig aanwezig.
- [ ] ITF-14 bearer frame is volledig zichtbaar.
- [ ] Geen barcodezone wordt door flex-shrink of overflow verplaatst.

### A6-carton

- [ ] `@page` is 105 x 148 mm.
- [ ] Labelbreedte is 85 mm.
- [ ] Het label heeft geen vaste hoogte die barcodezones samendrukt.
- [ ] GS1-128 en ITF-14 zijn volledig zichtbaar.
- [ ] De compactere tekst past zonder overlap of afkapping.

### A4-pallet

- [ ] Het palletlabel gebruikt A4 portrait.
- [ ] De 2x2-datagrid is volledig zichtbaar.
- [ ] De GS1-128-barcode staat volledig binnen de labelgrenzen.
- [ ] De footer en interface-elementen worden niet meegeprint.

### Fysieke printercontrole

- [ ] Test ten minste één printer die in de EUROstyle-logistiek wordt gebruikt.
- [ ] Controleer zwartniveau, contrast en scherpte van beide barcodes.
- [ ] Controleer dat papier niet automatisch naar een andere schaal wordt aangepast.
- [ ] Scan het fysieke printresultaat met de acceptatiescanner.
- [ ] Bewaar een PDF of foto van het geaccepteerde testresultaat bij de release.

## 5. Go/No-Go criteria

### Go

Een release krijgt Go wanneer:

- [ ] Alle P0-items die in scope zijn afgerond.
- [ ] Alle blokkerende CI-checks groen zijn.
- [ ] Barcode-regressietests groen zijn.
- [ ] Alle zes merk/mode-routes correct laden.
- [ ] JSON round-trip en oude JSON-import zijn getest.
- [ ] Scannerchecklist volledig is afgetekend.
- [ ] Printerchecklist volledig is afgetekend.
- [ ] Geen open P0- of onverklaarde P1-bug bestaat.
- [ ] Rollbackcommit en verantwoordelijke beschikbaar zijn.
- [ ] Release owner en logistics key user akkoord geven.

### No-Go

De release wordt niet gepubliceerd wanneer:

- [ ] Een barcode niet scanbaar is of een verkeerd AI-resultaat geeft.
- [ ] PI=0/PI=1 of een check digit afwijkt.
- [ ] Een ondersteunde productie-URL de verkeerde mode of het verkeerde merk toont.
- [ ] Een bestaand JSON-bestand data verliest.
- [ ] A4/A6 quiet zones of labelgrenzen niet kloppen.
- [ ] CI een relevante check overslaat of alleen met `continue-on-error` slaagt.
- [ ] Er geen bewezen rollbackpad beschikbaar is.
- [ ] De logistics key user niet beschikbaar is voor de afgesproken acceptatie.

### Voorwaardelijke release

Een voorwaardelijke release is alleen toegestaan voor documentatie-, console- of niet-functionele P3-punten. De uitzondering wordt schriftelijk vastgelegd met eigenaar en deadline. Voor barcode, routing, JSON of fysieke printkwaliteit bestaat geen voorwaardelijke acceptatie.

## 6. Release checklist

### Scope en code

- [ ] Release-scope en gewijzigde backlogitems zijn bekend.
- [ ] Geen niet-gerelateerde wijzigingen staan in de releasecommit.
- [ ] Barcode- en printcontracten zijn niet onbedoeld gewijzigd.
- [ ] Alle gewijzigde assets bestaan en zijn bereikbaar vanuit hun route.

### Tests

- [ ] Unit-/domeintests groen.
- [ ] HTML-, JavaScript- en assetvalidatie groen.
- [ ] Browser-/route-tests groen.
- [ ] JSON-import/export-tests groen.
- [ ] Scannerchecklist groen.
- [ ] Printerchecklist groen.

### Deployment

- [ ] Release-tag is aangemaakt.
- [ ] Known-good rollbackcommit is geregistreerd.
- [ ] Deploymentartifact bevat `index.html`, palletbestanden, `404.html`, `CNAME` en logo-assets.
- [ ] GitHub Pages-deployment is geslaagd.
- [ ] Custom domain reageert via HTTPS.
- [ ] Cache- en propagatierisico zijn bekend.

### Communicatie

- [ ] Changelog is bijgewerkt.
- [ ] Release owner is geinformeerd.
- [ ] Logistics key user is geinformeerd.
- [ ] Productievenster en supportcontact zijn bekend.
- [ ] Rollbackbesluitvormer is bereikbaar.

## 7. Productie validatie checklist

Voer deze checklist uit nadat de productie-URL is bijgewerkt en voordat de release als voltooid wordt gemarkeerd.

### Route en assets

- [ ] `https://labels.eurostyle.nl/ecostyle` opent ECOstyle carton.
- [ ] `https://labels.eurostyle.nl/vitalstyle` opent VITALstyle carton.
- [ ] `https://labels.eurostyle.nl/azstyle` opent AZ STYLE carton.
- [ ] De drie pallet-URL's openen palletlabels met het juiste merk.
- [ ] Logo's laden zonder assetfouten.
- [ ] `bwip-js` laadt en genereert barcodes.
- [ ] Ongeldige routes tonen de afgesproken foutpagina.

### Cartonlabel

- [ ] Referentie-EAN, product, aantal en batch worden correct voorgeladen.
- [ ] GS1-128 en ITF-14 zijn zichtbaar.
- [ ] THT/prod-label gedraagt zich volgens de mission.
- [ ] A4-preview en A6-preview zijn correct.
- [ ] Save/open werkt met een nieuw exportbestand.
- [ ] Save/open werkt met minimaal één oud productie-exportbestand.

### Palletlabel

- [ ] Referentie-EAN, count, PROD en batch worden correct voorgeladen.
- [ ] Alleen GS1-128 wordt getoond.
- [ ] De 2x2-grid toont Content, Count, PROD en Batch.
- [ ] PROD-conversie naar de GS1-128-waarde is correct.
- [ ] Save/open werkt met nieuw en oud palletformaat.

### Fysieke controle

- [ ] Eén cartonlabel op A4 is geprint en gescand.
- [ ] Eén cartonlabel op A6 is geprint en gescand.
- [ ] Eén palletlabel op A4 is geprint en gescand.
- [ ] Scanneroutput is opgeslagen bij de release-evidence.
- [ ] Productievalidatie is door de logistics key user afgetekend.

## 8. Noodprocedure bij regressies

### Herkenning

Start de noodprocedure bij één van deze signalen:

- barcode is niet scanbaar;
- scanner geeft verkeerde AI's of waarden;
- een productie-URL toont een verkeerd merk of mode;
- labels worden verkeerd geschaald of afgesneden;
- bestaande JSON-import werkt niet meer;
- logo's of `bwip-js` ontbreken op productie;
- meerdere gebruikers melden hetzelfde productieprobleem.

### Eerste 15 minuten

1. Stop verdere releases en zet de release owner op de hoogte.
2. Noteer URL, tijdstip, browser, merk, mode, invoerdata en printer/scanner.
3. Bewaar een foutieve PDF/foto en scanneroutput; wijzig het testlabel niet.
4. Controleer of het probleem reproduceerbaar is op de known-good versie.
5. Beslis of de fout labelproductie onveilig maakt.

### Directe mitigatie

- Bij barcode-, route-, JSON- of printfout: publicatie pauzeren en rollback uitvoeren.
- Bij alleen CDN-probleem: gebruik de gedocumenteerde fallback indien beschikbaar; anders rollback.
- Bij één lokale printer: markeer die printer als tijdelijk ongeschikt en controleer met de referentieprinter.
- Gebruikers mogen geen label met onbekende scanstatus vrijgeven voor logistiek gebruik.

### Rollback en verificatie

1. Publiceer de laatste known-good release.
2. Controleer de productie-URL's en assets.
3. Voer minimaal één carton- en één pallet-scan uit.
4. Voer minimaal één A4-printcontrole uit; voeg A6 toe wanneer de fout carton/A6-gerelateerd was.
5. Communiceer duidelijk dat de vorige versie is hersteld.
6. Maak een incidentticket met impact, oorzaak, tijdlijn, rollback en vervolgactie.

### Herstel na rollback

- Nieuwe fixes worden eerst op een aparte branch en testlocatie gevalideerd.
- De oorspronkelijke release wordt niet opnieuw gepubliceerd zonder aanvullende test.
- Het incident krijgt een nieuw backlogitem met passende prioriteit.
- Een herrelease vereist opnieuw de volledige Go/No-Go-beslissing.

## Fase-governance per prioriteitsniveau

De volgende blokken gelden naast de concrete backlogitems. Een fase mag pas worden afgesloten wanneer alle voorwaarden in het bijbehorende blok zijn afgetekend.

### P0 Kritiek

- **Doel:** Directe productie-, barcode-, route- en deploymentrisico's elimineren zonder bestaand gedrag te veranderen.
- **Risico:** Een snelle fix kan een andere merkvariant, JSON-variant of printroute breken.
- **Rollback methode:** Herstel de laatste known-good commit; bij een geïsoleerde asset- of JSON-fix mag alleen worden teruggedraaid als de volledige vorige artifactversie beschikbaar is.
- **Acceptatiecriteria:** Alle P0-items zijn opgelost; referentiebarcodes zijn exact gelijk; alle productie-URL's laden; oude JSON-bestanden blijven leesbaar; CI blokkeert relevante fouten.
- **Benodigde tests:** Barcode-unittests, route smoke tests, asset existence tests, JSON round-trip, drie merkvarianten, carton en pallet, minimaal één scannercontrole.
- **Stopcriteria:** Eén onscanbare barcode, dataverlies, verkeerde mode/brand, ontbrekende asset of niet-blokkerende CI-fout stopt de fase.

### P1 Hoog

- **Doel:** Routing, invoervalidatie, printcontract, gedeelde barcodekern en JSON-schema voorspelbaar maken.
- **Risico:** Structurele wijzigingen raken tegelijk URL's, barcodeoutput en bestaande exports.
- **Rollback methode:** Release terug naar de vorige bekende artifactversie; migraties moeten read-only backward compatible zijn voordat exportformaat verandert.
- **Acceptatiecriteria:** Eén routingstrategie is gedocumenteerd; ongeldige input wordt geweigerd; A4/A6 blijven scanbaar; één centrale GS1-bron produceert dezelfde waarden; oud en nieuw JSON-formaat werken.
- **Benodigde tests:** Volledige domeinregels, route-matrix, A4/A6-visuele tests, barcode-regressies, JSON migratietests, scannerchecklist en printerchecklist.
- **Stopcriteria:** Onverklaarde outputverschillen, een niet-migreerbaar JSON-formaat, quiet-zoneverlies of een route die afhankelijk is van toevallige URL-substrings.

### P2 Middel

- **Doel:** Duplicatie verwijderen, verantwoordelijkheden scheiden, regressiedekking uitbreiden en performance/security verbeteren.
- **Risico:** Refactoring kan bestaand gedrag wijzigen terwijl de code er op papier correct uitziet.
- **Rollback methode:** Werk in kleine, afzonderlijk deploybare commits; rollback naar de laatste commit waarvan browser-, scanner- en printertests groen zijn.
- **Acceptatiecriteria:** Merk en mode zijn configuratie; domeinlogica is centraal en testbaar; browserchecks dekken alle routes; logo-import en dependency-loading zijn beheerst.
- **Benodigde tests:** Unit-, integratie-, e2e-, visuele-, performance- en dependency/foutpadtests; volledige scanner- en printerregressie bij wijzigingen aan rendering.
- **Stopcriteria:** Meer dan één bron voor dezelfde GS1-regel blijft actief zonder expliciete compatibility-reden, of CI kan de nieuwe structuur niet betrouwbaar testen.

### P3 Laag

- **Doel:** Opruimen, documenteren, tooling reproduceerbaar maken en gebruikservaring verbeteren zonder productiecontracten te wijzigen.
- **Risico:** Lage prioriteit kan ten onrechte leiden tot wijzigingen aan stabiele barcode- of printcode.
- **Rollback methode:** Individuele kleine commits terugdraaien; productie-rollback is alleen nodig wanneer P3 toch runtimegedrag raakt.
- **Acceptatiecriteria:** Geen ongebruikte productiecode zonder reden; documentatie beschrijft de actuele werking; toolingversies zijn reproduceerbaar; foutstatussen zijn bruikbaar.
- **Benodigde tests:** Bestaande regressiesuite, lint/documentatiecontrole, toegankelijkheidschecks en smoke tests.
- **Stopcriteria:** Een P3-wijziging verandert barcodebytes, routegedrag, JSON-compatibiliteit of fysieke printoutput zonder dat het item opnieuw naar P1 wordt geprioriteerd.