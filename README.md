# Recipe Binder PWA — Setup Guide

Your recipe binder is now a full Progressive Web App. Once installed on your iPhone, it works offline, has its own home screen icon, and runs full-screen like a native app.

## What's in this folder

- `index.html` — the app itself
- `manifest.webmanifest` — PWA configuration
- `sw.js` — service worker (enables offline mode)
- `icons/` — app icons for home screen

All four pieces need to be hosted together on the same URL.

## Deploy in ~60 seconds (recommended: Netlify Drop)

Netlify Drop is the fastest free host and gives you a live URL immediately.

1. Go to **https://app.netlify.com/drop**
2. Drag this entire folder (not the zip — unzip first) onto the page
3. Wait ~5 seconds — you'll get a URL like `https://lucky-cat-abc123.netlify.app`
4. (Optional) Sign up for a free account to claim the site and rename the URL

**Important:** PWAs require HTTPS. Netlify gives you HTTPS automatically. Do not host this via plain iCloud file links — the service worker won't register.

## Install on your iPhone

1. Open the Netlify URL in **Safari** (must be Safari, not Chrome)
2. Tap the Share button (square with arrow pointing up)
3. Scroll down and tap **"Add to Home Screen"**
4. Name it (e.g., "Recipes") and tap Add

Now you have a "Recipes" icon on your home screen. Tap it — it opens full-screen with no Safari UI, just like a real app.

## What works offline

After your first visit, the entire app is cached locally:
- Browsing all recipes ✅
- Scaling servings ✅
- Step checkboxes ✅
- Timers ✅
- Save as PDF ⚠️ (needs a brief online moment the first time — the PDF print dialog uses browser print; it works offline after that)

## Alternative hosts

- **GitHub Pages** — more permanent, needs a GitHub account and a bit more setup
- **Cloudflare Pages** — similar to Netlify, also free
- **Vercel** — also free, drag & drop

Avoid: iCloud Drive direct links, Dropbox share links, Google Drive — none of these provide proper HTTPS for service workers.

## Updating the binder

When you want to add new recipes:
1. Ask Claude to rebuild the PWA with the new recipes
2. Replace `index.html` in your hosted folder with the new version
3. On your iPhone, close the app fully and reopen — the service worker will pull the update

The version number in `sw.js` (`CACHE_NAME = "recipe-binder-v1"`) should be bumped (`v2`, `v3`...) whenever you update `index.html`, to force the cache to refresh. Claude will handle this automatically when rebuilding.
