# Deployment & Infrastructure

> **Actuele bron:** GitHub Pages publiceert vanaf `gh-pages` met folder `/` (root). Oude tekst waarin `main` als Pages-source wordt genoemd is historische informatie en geldt niet als actuele configuratie.

## Project Location

**Local Network Path (Development):**
```
\bpa01\c$\_EUROstyleTools\EUROstyleScripts\AI\itf14_gs1_128_label_generator
```

**GitHub Repository:**
```
https://github.com/EUROstyle-BV/itf14_gs1_128_label_generator
```

---

## Live URLs

The application is publicly hosted at:

### Production Domain
- **Base:** `https://labels.eurostyle.nl/`
- **ECOstyle:** `https://labels.eurostyle.nl/ecostyle`
- **VITALstyle:** `https://labels.eurostyle.nl/vitalstyle`
- **AZ STYLE:** `https://labels.eurostyle.nl/azstyle`

### GitHub Pages (Fallback)
- **ECOstyle:** `https://eurostyle-bv.github.io/itf14_gs1_128_label_generator/ecostyle`
- **VITALstyle:** `https://eurostyle-bv.github.io/itf14_gs1_128_label_generator/vitalstyle`
- **AZ STYLE:** `https://eurostyle-bv.github.io/itf14_gs1_128_label_generator/AZstyle`

---

## Hosting Setup

### Custom Domain (`labels.eurostyle.nl`)

The custom domain `labels.eurostyle.nl` is configured as a **CNAME** pointing to:
```
eurostyle-bv.github.io
```

**GitHub Pages Settings:**
- Repository: `EUROstyle-BV/itf14_gs1_128_label_generator`
- Custom domain: `labels.eurostyle.nl`
- Source: **Deploy from a branch**
- Branch: `gh-pages`
- Folder: `/` (root)
- HTTPS: Enabled via GitHub Pages when certificate status is approved

**SPA Routing:**
- `404.html` performs client-side URL rewriting for brand variant routing
- `pathSegmentsToKeep = 0` (no repository name prefix with custom domain)
- URL pattern: `/?/{brand}` is converted to `/?/ecostyle`, `/?/vitalstyle`, etc.

---

## Deployment Workflow

### 1. Local Development
```bash
# Clone from GitHub
git clone https://github.com/EUROstyle-BV/itf14_gs1_128_label_generator.git

# Work on index.html and other files
# No build step required — test directly in browser

# Test all brand variants
# - Open index.html locally
# - Or access via live URLs above
```

### 2. Push to GitHub
```bash
# Stage changes
git add .

# Commit with descriptive message
git commit -m "description of changes"

# Push to main
git push origin main
```

### 3. GitHub Pages Deployment
After pushing to `main`, GitHub Actions validates the repository and publishes a temporary `deploy/` directory to `gh-pages`. GitHub Pages then serves the root of `gh-pages`.

The custom domain is updated only after the Pages build completes. Propagation and CDN caching can delay visible changes.

---

## File Accessibility

### From Network Share
All team members can access the project via the network share:
```
\bpa01\c$\_EUROstyleTools\EUROstyleScripts\AI\itf14_gs1_128_label_generator
```

### From GitHub
- Clone: `git clone https://github.com/EUROstyle-BV/itf14_gs1_128_label_generator.git`
- Web: `https://github.com/EUROstyle-BV/itf14_gs1_128_label_generator`

---

## Key Files for Deployment

| File | Purpose |
|---|---|
| `index.html` | Carton label generator (single file, all-in-one) |
| `pallet.html` | Pallet label generator (single file, all-in-one) |
| `404.html` | GitHub Pages SPA routing redirect |
| `images/Logo_*.jpg` | Brand logos |
| `mission.md` | Technical specification (reference only) |
| `CLAUDE.md` | Developer notes (reference only) |
| `DEPLOYMENT.md` | This file (deployment info) |
| `README.md` | User-facing documentation |

---

## Troubleshooting Deployment

| Issue | Solution |
|---|---|
| Custom domain not resolving | Check CNAME record: `labels.eurostyle.nl → eurostyle-bv.github.io`. GitHub DNS may take up to 48 hours to fully propagate. |
| Brand variants not loading | Verify `404.html` is in repository root and HTTPS is enabled in GitHub Pages settings. |
| Barcodes not rendering on live URL | Verify bwip-js CDN is accessible (check browser console for errors). |
| Recent changes not visible | Hard refresh browser (Ctrl+Shift+R or Cmd+Shift+R) to clear cache. GitHub Pages cache may take up to 5 minutes. |

---

## Maintenance & Updates

- **No server required** — pure static hosting via GitHub Pages
- **No build step** — edit `index.html` directly
- **Continuous deployment** — push to `main` and changes go live automatically
- **Backup:** The network share at `\bpa01\c$\_EUROstyleTools\EUROstyleScripts\AI` provides a local backup of the latest code

---

## Security Notes

- ✅ No authentication required for end users
- ✅ No server-side code execution
- ✅ All barcode data processed client-side (browser)
- ✅ No external API calls except CDN (bwip-js)
- ✅ HTTPS enforced via GitHub Pages
- ✅ No data persistence on servers

---

**Version:** 2.0
**Last Updated:** September 2026
