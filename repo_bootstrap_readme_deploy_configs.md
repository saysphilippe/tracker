# Dependency Tracker — README

A single‑page React app to track cross‑project dependencies. Includes local storage persistence, status change logging, CSV export, filtering/sorting, and a clean UI with Tailwind + lucide-react.

---

## 1) Clone & bootstrap
```bash
# Replace <brukernavn> with your GitHub handle
git clone https://github.com/<brukernavn>/dependency-tracker.git
cd dependency-tracker

# Create a Vite React app in the current folder
npm create vite@latest . -- --template react
npm install

# Icons used by the app
npm install lucide-react
```

> **Tip:** If du allerede har et eksisterende Vite-prosjekt, hopp over `npm create vite` og bruk mappestrukturen du har.

---

## 2) Legg inn app-koden
- Åpne `src/App.jsx` og **erstatt alt** innholdet med komponenten i "Dependency Tracker (React) — single‑file app" (canvasen ved siden av denne chatten).
- Sørg for at `src/main.jsx` importerer `./index.css` (standard i Vite‑malen).

---

## 3) Sett opp Tailwind CSS
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

**tailwind.config.js** – oppdater `content`:
```js
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: { extend: {} },
  plugins: [],
}
```

**src/index.css** – erstatt med:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## 4) Kjør lokalt
```bash
npm run dev
```
Vite viser en lokal URL (typisk `http://localhost:5173`).

---

## 5) Bygg
```bash
npm run build   # output til ./dist
npm run preview # valgfritt, for å teste prod-bygget lokalt
```

---

## 6) Deploy (Vercel – anbefalt)
Alternativ A – **Dashboard**:
1. Push repo til GitHub.
2. Gå til https://vercel.com/new → velg repoet.
3. Framework: **Vite**. Build command: `npm run build`. Output dir: `dist`.
4. Deploy – ferdig!

Alternativ B – **CLI**:
```bash
npm i -g vercel
vercel        # første gang – svar på spørsmålene
vercel --prod # prod-deploy
```

> **SPA routing?** Denne appen bruker ikke router; hvis du senere legger til React Router, bør du ha en catch‑all rewrite (se `vercel.json` under).

---

## 7) Netlify (valgfritt)
- App → *New site from Git* → velg repo.
- Build command: `npm run build`  •  Publish directory: `dist`.
- Legg til `netlify.toml` (under) for SPA‑fallback hvis du får 404 ved manuell URL‑oppdatering.

---

## 8) Cloudflare Pages (valgfritt)
- *Pages* → *Connect to Git* → velg repo.
- Build: `npm run build`  •  Output: `dist`.

---

## 9) CSV‑eksport
Knappen **Export CSV** laster ned `dependency_log.csv` med alle synlige felt + siste change‑log.

---

## 10) Struktur & persistens
- All data lagres i **LocalStorage** (`dependency-tracker:data:v1`).
- Opprett, rediger, slett direkte i UI. Statusendring logger automatisk i historikken.

---

## 11) Vanlige feil
- **Tom side / styling mangler** → Tailwind er ikke satt opp riktig (sjekk `tailwind.config.js` og `index.css`).
- **Ikoner mangler** → `lucide-react` er ikke installert.
- **404 på refresh** (med router) → legg til rewrite i `vercel.json` eller `netlify.toml` (se under).

---

## 12) Lisens
MIT (endre etter behov).

---

# vercel.json
```json
{
  "version": 2,
  "builds": [
    { "src": "package.json", "use": "@vercel/static-build", "config": { "distDir": "dist" } }
  ],
  "framework": "vite",
  "git": { "deployOnPush": true },
  "routes": [
    { "src": "/favicon.ico", "dest": "/favicon.ico" },
    { "src": "/assets/(.*)", "dest": "/assets/$1" },
    { "src": "/(.*)", "dest": "/index.html" }
  ]
}
```
> `routes`-delen er kun nødvendig hvis du bruker SPA‑routing (React Router). For denne appen uten router kan den beholdes eller fjernes – begge deler fungerer.

---

# netlify.toml (valgfritt)
```toml
[build]
  command = "npm run build"
  publish = "dist"

# Kun nødvendig for SPA‑routing (React Router). Ikke påkrevd for denne appen.
[[redirects]]
  from = "/api/*"
  to = "/api/:splat"
  status = 200

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

---

# .gitignore (Node)
```gitignore
# dependencies
/node_modules

# production
/dist
/build

# misc
.DS_Store
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*
.vercel
```

---

## Oppsummering
1. Lag repo → bootstrap med Vite → lim inn `App.jsx` fra canvas.
2. Installer `lucide-react` og sett opp Tailwind.
3. `npm run dev` lokalt → `npm run build` → deploy til Vercel/Netlify/CF Pages.

Gi meg repo‑URLen din om du vil at jeg sanity‑sjekker bygg og `vercel.json`. 



---

# GitHub Actions (CI)

Legg disse filene i repoet ditt under `.github/workflows/`.

## 1) `ci.yml` — bygg & artefakt på hver push/PR
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install deps
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload production artifact (dist/)
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist
```

> Krever at prosjektet bruker Vite/CRA med `npm run build` som lager `dist/` eller `build/`. Hvis du bruker CRA, endre `path: build`.

## 2) `vercel-deploy.yml` — automatisk deploy til Vercel (valgfritt)
```yaml
name: Deploy to Vercel

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build

      - name: Vercel Deploy
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: .
          vercel-args: '--prod'
```

**Slik får du hemmeligheter (Secrets):**
1. Opprett prosjektet i Vercel (eller importer repoet).
2. Hent `ORG_ID` og `PROJECT_ID` fra **Vercel → Settings → General** (kopier ID‑ene).
3. Lag en **Personal Token** i **Vercel → Account Settings → Tokens** og kall den f.eks. `github-ci`.
4. I GitHub‑repoet: **Settings → Secrets and variables → Actions → New repository secret** og legg til:
   - `VERCEL_TOKEN`
   - `VERCEL_ORG_ID`
   - `VERCEL_PROJECT_ID`

> Alternativt kan du bruke `vercel/cloudflare-deploy-action` eller Vercels offisielle `deploy-pr-comment`‑flow; denne varianten er enkel og stabil.

## 3) `netlify-deploy.yml` — deploy til Netlify (valgfritt)
```yaml
name: Deploy to Netlify

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build

      - name: Deploy
        uses: nwtgck/actions-netlify@v2.0
        with:
          publish-dir: ./dist
          production-branch: main
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: 'CI deploy'
          enable-commit-comment: false
          enable-pull-request-comment: false
          overwrites-pull-request-comment: false
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

**Netlify‑secrets:** I Netlify → User settings → Applications → Personal access tokens → lag token → sett som `NETLIFY_AUTH_TOKEN`. Finn `Site ID` under Site settings → General → Site details → `Site information`.

## 4) Lint/Test (valgfritt)
Legg til disse stegene i `ci.yml` hvis du har ESLint/Jest/Vitest:
```yaml
      - name: Lint
        run: npm run lint --if-present

      - name: Test
        run: npm test --if-present -- --ci
```

