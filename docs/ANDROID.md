# Turning Makari Islamic TV into an Android App (via GitHub)

This app is already a real, installable PWA (valid manifest, real icons,
working service worker). The standard, Google-recommended way to turn a
PWA into an Android app is a **TWA (Trusted Web Activity)** — a thin native
wrapper that launches your live website full-screen, no browser chrome.
Google's own tool for this is **Bubblewrap**, and there's a GitHub Actions
workflow already set up in this repo at
`.github/workflows/build-android.yml` to build it automatically.

Read this whole page before starting — a few steps are one-time and
**interactive**, so they can't be scripted for you sight-unseen. Skipping
or reordering them is the #1 cause of a broken TWA (shows a URL bar
instead of looking native, or Play Store rejects updates).

---

## Before you start — this must already be true

- [ ] `frontend/` is deployed at a **live, public HTTPS URL** (GitHub
      Pages, Render Static Site, Netlify, Vercel — any is fine). A TWA
      launches a real website; there is nothing to wrap until it's live.
- [ ] Visiting that URL in Chrome on Android shows an "Add to Home
      screen" / install prompt, and the app opens with no visible browser
      UI once installed. If that doesn't work yet, fix the PWA first —
      packaging a broken PWA just gives you a broken app.
- [ ] `backend/` is deployed and `ALLOWED_ORIGINS` includes your frontend's
      exact deployed origin (see backend README) — otherwise API calls
      from the installed app will fail with CORS errors.

**If you're using GitHub Pages specifically:** project sites are served at
`https://<user>.github.io/<repo>/`, a subpath — the manifest, service
worker, and icon paths in this repo are already written to work at any
subpath (relative paths, not `/absolute`), and `.nojekyll` files are
included at the repo root and in `frontend/` so GitHub Pages doesn't
silently strip the `.well-known/` folder (Jekyll excludes dot-folders by
default, which would break TWA verification with no obvious error).

---

## One-time setup (do this once, on your own machine)

You need [Node.js](https://nodejs.org) and a JDK installed locally.

```bash
npm install -g @bubblewrap/cli
cd makari-islamic-tv
bubblewrap init --manifest https://YOUR-DEPLOYED-URL/manifest.json
```

This asks you interactively for:
- **Application ID** (e.g. `com.makari.islamictv` — reverse-domain style,
  can't be changed later without becoming a different app on Play Store)
- **Signing key details** — Bubblewrap generates a new keystore
  (`android.keystore`) and asks you to set passwords. **Save these
  somewhere safe** (a password manager). Losing this keystore means you
  can never publish an update to the same app listing again — there is no
  recovery.

When it finishes, you'll have:
- `twa-manifest.json` — commit this to the repo root
- `android.keystore` — **do NOT commit this to git.** Instead:

```bash
base64 -i android.keystore -o keystore.b64   # Linux/macOS
# Windows PowerShell: certutil -encode android.keystore keystore.b64
```

Then in GitHub: **Settings → Secrets and variables → Actions → New repository secret**, add:

| Secret name | Value |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | contents of `keystore.b64` |
| `ANDROID_KEYSTORE_PASSWORD` | the keystore password you set |
| `ANDROID_KEY_ALIAS` | the key alias you set (often `android`) |
| `ANDROID_KEY_PASSWORD` | the key password you set |

## Get your signing fingerprint and finish TWA verification

```bash
keytool -list -v -keystore android.keystore -alias YOUR_ALIAS
```

Copy the `SHA256:` fingerprint it prints, then edit
`frontend/.well-known/assetlinks.json` in this repo — replace
`REPLACE_WITH_YOUR_APK_SHA256_FINGERPRINT` with that value (keep the
colons, uppercase, exactly as printed) and `com.makari.islamictv` with
whatever Application ID you actually chose. Redeploy the frontend so
`https://YOUR-DEPLOYED-URL/.well-known/assetlinks.json` is live and
returns that JSON (check it loads directly in a browser — a 404 or HTML
error page there is the single most common reason a TWA still shows a
URL bar).

Commit `twa-manifest.json` and push.

---

## Building the app (this part IS automated)

Once `twa-manifest.json` is committed and the four secrets above are set,
push to `main` (or go to **Actions → Build Android App (TWA) → Run
workflow**). It will:
1. Install Bubblewrap + a JDK
2. Restore your keystore from the secret
3. Build a signed `.aab` (for Play Store) and `.apk` (for direct install/testing)
4. Upload both as a downloadable workflow artifact

Every time you update the PWA (frontend changes), just re-run the
workflow — no need to redo the one-time setup.

---

## Installing it on your phone right now (no Play Store needed)

Download the `.apk` from the workflow's Artifacts, transfer it to your
phone, and open it (you'll need to allow "install from unknown sources"
once). This is the fastest way to test — it's the real signed app, not a
preview.

## Publishing to the Play Store (optional, needs a Google account)

Upload the `.aab` from the workflow artifact to the [Play Console](https://play.google.com/console)
(one-time $25 developer registration fee). Google will run its own
automated review — this is outside what any tooling here can pre-verify.

---

## Common failure points and what they mean

| Symptom | Cause |
|---|---|
| App shows a URL/address bar instead of full-screen | `assetlinks.json` isn't live yet, has the wrong fingerprint, or isn't served with `Content-Type: application/json` |
| `bubblewrap init` fails to fetch the manifest | Frontend isn't deployed yet, or isn't HTTPS |
| Workflow fails at "Restore signing keystore" | A secret is missing/misspelled — check exact names above |
| Installed app can't log in / load lectures | Backend `ALLOWED_ORIGINS` doesn't include the frontend's deployed origin — CORS blocks it |
| App installs but shows blank/broken pages | Same subpath issue this repo's relative-path fixes address — confirm you deployed the whole `frontend/` folder as-is, including `.well-known/` and `icons/` |
