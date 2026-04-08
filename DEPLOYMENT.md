# Deployment Guide

## Public Hosting (Optie 1 - Recommended)

Het label generator is nu voorbereidt voor openbare hosting op een eigen domein.

### Setup: `labels.eurostyle.nl`

1. **Copy `index.html` naar webserver**
   - Geen build stap nodig
   - Geen dependencies nodig
   - Kan rechtstreeks op CDN/hosting platform

2. **CORS & Security Headers** (optioneel)
   ```
   Access-Control-Allow-Origin: *
   X-Content-Type-Options: nosniff
   X-Frame-Options: SAMEORIGIN
   ```

3. **DNS Setup**
   ```
   labels.eurostyle.nl → je hosting server IP
   ```

### Resultaat

- ✅ Klanten kunnen direct naar `https://labels.eurostyle.nl` gaan
- ✅ Geen login/wachtwoord management
- ✅ Minimale support overhead
- ✅ Subtiele "Powered by EUROstyle" footer

---

## Optional: Analytics (Toekomstig)

As template voor wanneer je analytics wilt inschakelen:

```javascript
// Uncomment de volgende rule in index.html (zoek: "logPageView()")
logPageView();
```

Dit stuurt simpele page view events naar een analytics endpoint.
Geen cookies, geen tracking van andere data.

---

## Optional: A/B Testing

In toekomst kun je dit gebruiken voor feature A/B testing:

```javascript
const variant = Math.random() > 0.5 ? 'A' : 'B';
// Toon andere UI/features op basis van variant
```

---

## Hosting Opties

| Platform | Setup | Prijs | Notes |
|---|---|---|---|
| **GitHub Pages** | Enable pages from main branch root | Free | Static host, perfect for static HTML |
| **Netlify** | Drag-drop index.html | Free | Fast CDN, SSL included |
| **Vercel** | Git deploy | Free | Built for modern web, overkill maar snel |
| **EUROstyle VPS** | SCP upload | Your server | Full controle, moet je zelf HTTPS configuren |

**Aanbeveling:** GitHub Pages als je snelle, eenvoudige public hosting wilt; Netlify/Vercel als je extra CDN of build pipelines nodig hebt.

---

## GitHub Pages Setup

Voor deze repo is GitHub Pages de eenvoudigste manier om `index.html` te publiceren.

1. Zorg dat `index.html` in de root van de repository staat.
2. Ga naar je repository op GitHub.
3. Open `Settings` → `Pages`.
4. Stel de source in op branch `main` en folder `/ (root)`.
5. Klik op `Save`.
6. Wacht enkele minuten totdat GitHub Pages de site heeft gepubliceerd.

Je site wordt dan bereikbaar op:

```
https://<gebruikersnaam>.github.io/<repositorynaam>/
```

### Custom domain (optioneel)

Als je een eigen domein wilt gebruiken, kun je:

- een `CNAME` bestand toevoegen met je domeinnaam, of
- de custom domain sectie in GitHub Pages settings gebruiken.

Voor een custom subdomein zoals `labels.eurostyle.nl` voer je de DNS records in je domeinbeheer in en voeg je dezelfde domeinnaam toe in GitHub Pages settings.

---

---

## Security Notes

Deze app **slaat GEEN gegevens op** — alles gebeurt in browser geheugen.
- No server-side data storage
- No cookies
- Alles wordt gewist als pagina sluit

**Veilig voor public hosting** ✅

---

## Monitor & Analytics

Zodra live, kun je gebruiken:
- **Google Analytics** (gratis, privacy-friendly)
- **Sentry** (error tracking, free tier)
- **Netlify Analytics** (als je Netlify gebruikt)

Allemaal optioneel — app werkt prima zonder.

---

## Toekomst: Login/Branding

Mocht je later toch login willen toevoegen:
1. Add simple auth layer (Clerk, Auth0, Firebase)
2. Track per-user usage
3. Voeg klant-logo in header in

Maar voor nu: **keep it simple & public**. 🚀
