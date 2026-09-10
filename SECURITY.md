# Security

## Scope

Deze applicatie is een statische client-side labelgenerator. Er is geen backend, database, login of server-side labelopslag.

## Secrets

- Commit geen tokens, wachtwoorden, private keys of persoonlijke gegevens.
- Gebruik GitHub Secrets voor eventuele toekomstige credentials.
- Plaats geen scannerdata, klantdata of productiegegevens in debuglogs.
- Controleer exports voordat ze buiten de lokale gebruiker worden gedeeld.

## GitHub Actions

De deploymentworkflow gebruikt momenteel:

- `contents: write` voor publiceren naar `gh-pages`;
- `actions/checkout`;
- `actions/setup-node`;
- `JamesIves/github-pages-deploy-action`.

Gebruik least privilege. Een toekomstige artifact-gebaseerde Pages-workflow zou alleen de noodzakelijke `pages: write` en `id-token: write` permissions moeten krijgen, naast read-only checkoutrechten.

## Dependencies

- `bwip-js` wordt extern via jsDelivr geladen;
- Google Fonts worden extern geladen;
- validatietools worden in CI geïnstalleerd.

Risico's:

- CDN-uitval;
- gewijzigde externe content;
- niet-reproduceerbare npm-versies;
- supply-chain impact.

Hardening:

- pin runtime dependencies;
- gebruik Subresource Integrity waar praktisch;
- overweeg lokaal vendoren van bedrijfskritische barcodecode;
- voeg een lockfile toe voor CI-tools;
- pin GitHub Actions op volledige commit-SHA's in plaats van alleen tags.

## Client-side imports

JSON en logo's worden lokaal door de browser gelezen. Valideer bestandstype, bestandsgrootte en JSON-structuur voordat toekomstige wijzigingen import uitbreiden. Sla bestanden niet server-side op zonder een nieuwe privacy- en securityanalyse.

## Meldingen

Securityproblemen worden niet publiek gemaakt voordat impact en herstel zijn beoordeeld. Meld een vermoedelijk securityprobleem aan de repository-eigenaar via het afgesproken private GitHub securitykanaal. Voeg geen secrets of persoonsgegevens toe aan een issue of pull request.
