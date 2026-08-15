# Turning Makari Islamic TV into an Android App (via GitHub)

**Status for this repo: the one-time setup described below is already
done.** The frontend is deployed at `https://makarii.onrender.com`, a real
signing keystore has been generated, `twa-manifest.json` is committed, and
`frontend/.well-known/assetlinks.json` has the real fingerprint. All that's
left is adding 3 secrets to GitHub and running the workflow — see
**"Finish the setup"** below.

---

## How this works

This app is a real, installable PWA (valid manifest, real icons, working
service worker). The standard way to turn a PWA into an Android app is a
**TWA (Trusted Web Activity)** — a thin native wrapper that launches your
live website full-screen, no browser chrome. Google's own tool for this is
**Bubblewrap**. `.github/workflows/build-android.yml` runs it automatically
using Google's official prebuilt container image, so nothing needs to be
installed locally to build.

---

## Finish the setup (3 steps, ~5 minutes)

### 1. Add these 3 repository secrets

GitHub → this repo → **Settings → Secrets and variables → Actions → New
repository secret**:

| Secret name | Value |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | *(given to you separately — see below)* |
| `ANDROID_KEYSTORE_PASSWORD` | *(given to you separately — see below)* |
| `ANDROID_KEY_PASSWORD` | *(same value as `ANDROID_KEYSTORE_PASSWORD`)* |

The key alias (`makari`) is already in `twa-manifest.json`, so it doesn't
need to be a secret.

**Save the keystore password somewhere safe (a password manager) even
after pasting it into GitHub.** If you ever need to build outside this
workflow (e.g. Android Studio) you'll need it again, and there is no way
to recover it if lost — losing it means you can never publish an update
to the same app listing again.

### 2. Confirm `assetlinks.json` is live

Visit `https://makarii.onrender.com/.well-known/assetlinks.json` in a
browser. It should show JSON (not a 404 or an HTML error page) containing
`"package_name": "com.makari.islamictv"` and a `sha256_cert_fingerprints`
value. If Render hasn't redeployed since this file was added, trigger a
manual deploy first.

### 3. Run the workflow

GitHub → **Actions → Build Android App (TWA) → Run workflow**. It will:
1. Pull Google's official Bubblewrap container (JDK + Android SDK
   pre-installed — this avoids a known issue where Bubblewrap's own
   installer prompts interactively even in CI, see
   [bubblewrap#806](https://github.com/GoogleChromeLabs/bubblewrap/issues/806))
2. Restore the signing keystore from your secret
3. Regenerate the Android project from `twa-manifest.json`
   (`bubblewrap update`)
4. Build a signed `.aab` (for Play Store) and `.apk` (for direct
   install/testing)
5. Upload both as a downloadable workflow artifact

Every time the PWA changes, just re-run the workflow — no need to redo
any of the above.

---

## Installing it on your phone right now (no Play Store needed)

Download the `.apk` from the workflow's Artifacts, transfer it to your
phone, and open it (Android will ask you to allow "install from unknown
sources" once). This is the real signed app, not a preview.

## Publishing to the Play Store (optional, needs a Google account)

Upload the `.aab` artifact to the [Play Console](https://play.google.com/console)
(one-time $25 developer registration fee). Google runs its own automated
review after that — outside anything this tooling can pre-verify.

---

## Common failure points and what they mean

| Symptom | Cause |
|---|---|
| App shows a URL/address bar instead of full-screen | `assetlinks.json` isn't live yet, has the wrong fingerprint, or isn't served with `Content-Type: application/json` — recheck step 2 above |
| Workflow fails at "Check twa-manifest.json exists" | The file was removed/renamed from the repo root |
| Workflow fails at "Restore signing keystore" | A secret is missing or misspelled — check the exact names above |
| Workflow fails at "Regenerate Android project" | `webManifestUrl`/`iconUrl` in `twa-manifest.json` aren't reachable — confirm `https://makarii.onrender.com/manifest.json` loads in a browser |
| Installed app can't log in / load lectures | Backend `ALLOWED_ORIGINS` doesn't include `https://makarii.onrender.com` — CORS blocks it. Check the backend service's environment variables |
| App installs but shows blank/broken pages | Confirm Render's Publish Directory is exactly `frontend` and the whole folder (including `.well-known/` and `icons/`) deployed |

---

## If you ever need to change the app (new package name, new domain, etc.)

Edit `twa-manifest.json` directly and re-run the workflow — it regenerates
the Android project from that file every time. You do **not** need to
regenerate the keystore for normal changes; only generate a new one if you
are deliberately starting a brand-new, unrelated app listing (a new
keystore means a new, unrelated Play Store app — it cannot update the old
one).
