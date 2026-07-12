# Yellowstone & Grand Teton — July 17–26, 2026

Family road trip itinerary. Installable on phones, works fully offline (there's almost no cell signal in the parks).

---

## Publishing this (GitHub Pages)

**One-time setup, ~3 minutes. No git or command line needed.**

1. Go to **github.com/new**
   - Repository name: `yellowstone-2026` (or anything)
   - Set it **Public** — Pages needs this on free accounts
   - Don't tick "Add a README" — we already have one
   - Click **Create repository**

2. On the next screen click **uploading an existing file**

3. Drag in **all 6 files** from this folder:
   ```
   index.html          ← the itinerary
   manifest.json       ← makes it installable
   sw.js               ← makes it work offline
   icon-192.png
   icon-512.png
   apple-touch-icon.png
   ```
   Click **Commit changes**

4. Go to **Settings** (top of repo) → **Pages** (left sidebar)
   - Source: **Deploy from a branch**
   - Branch: **main**, folder: **/ (root)**
   - Click **Save**

5. Wait ~1 minute. Your URL will be:
   ```
   https://YOUR-USERNAME.github.io/yellowstone-2026/
   ```
   Share that link with everyone. **It never changes.**

---

## Updating it later

Repo → click `index.html` → pencil icon (✏️) → paste the new version → **Commit changes**.
Live in about a minute. Everyone's link keeps working; no need to re-share.

**Important:** if you change `index.html`, also bump the cache version in `sw.js`:
```js
const CACHE = 'trip-v1';   →   const CACHE = 'trip-v2';
```
Otherwise phones that already saved it will keep showing the old copy.

---

## Telling everyone to save it offline

The page shows a banner with instructions, but to be explicit:

**iPhone (Safari — must be Safari, not Chrome):**
Open the link → tap **Share ⬆︎** → **Add to Home Screen** → **Add**

**Android (Chrome):**
Open the link → tap **⋮** → **Add to Home screen** (or the **Install** button in the banner)

It then behaves like an app: full screen, its own icon, opens with zero signal.

---

## What still needs a signal

Everything in the itinerary works offline — timelines, notes, closures, all of it.

**These do not:**
- The embedded map previews (they'll show blank)
- The "Open in Google Maps" buttons

**So everyone should also download offline maps before leaving:**
Google Maps app → profile picture → **Offline maps** → **Select your own map** → zoom to cover Yellowstone + Grand Teton → **Download**

Do the same for the Twin Falls / Idaho Falls corridor if you want turn-by-turn on the drive up.
