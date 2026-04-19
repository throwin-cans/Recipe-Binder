# Recipe Binder — Self-Updating PWA

Your binder now has an **Admin page** with a "Save to GitHub" button. Paste a recipe, tap save, done. No GitHub app, no manual file editing.

---

## How it works

- `index.html` — the main app (loads recipes from `recipes.json`)
- `admin.html` — hidden admin page that commits new recipes directly to GitHub
- `recipes.json` — your recipe data (the file the admin page edits)
- `sw.js` — service worker (offline support + always-fresh recipe data)
- `manifest.webmanifest` + `icons/` — PWA configuration

The admin page uses the GitHub REST API with a Personal Access Token (PAT) that you generate. The token lives in your browser only — it's never sent anywhere except GitHub itself.

---

## Step 1: Upload to GitHub Pages

1. In your `recipe-binder` GitHub repo, upload everything from this folder (same steps as before — unzip, upload contents of the folder into the repo root, commit).
2. Make sure GitHub Pages is enabled (Settings → Pages → Source: Deploy from branch: main / root).
3. Wait ~1 min. Your site is live at `https://YOUR-USERNAME.github.io/recipe-binder/`.

---

## Step 2: Create your Personal Access Token

This is the one-time setup that lets the admin page commit to your repo.

1. Go to **https://github.com/settings/personal-access-tokens/new** (this is the newer, safer "fine-grained" token page)
2. **Token name:** `recipe-binder-admin`
3. **Expiration:** pick something reasonable — 1 year is fine. You'll be prompted to renew when it expires.
4. **Resource owner:** your own account
5. **Repository access:** select **"Only select repositories"** and choose your `recipe-binder` repo. *Do not give it access to all repos.*
6. **Permissions** → Under **"Repository permissions"**, find **"Contents"** and set it to **Read and write**. Leave everything else alone.
7. Click **Generate token** at the bottom.
8. **Copy the token immediately** — it starts with `github_pat_...`. You won't see it again.

The token lets the admin page commit files to *only* the recipe-binder repo, and nothing else. If you ever lose the device or want to revoke it, go back to GitHub settings → Personal access tokens → delete it.

---

## Step 3: First-time admin setup on your phone

1. On your phone, open `https://YOUR-USERNAME.github.io/recipe-binder/admin.html` in Safari
2. First screen: enter your GitHub username, repo name (`recipe-binder`), branch (`main`), and create an admin password
3. Second screen: paste your PAT
4. You're in! The page will remember your settings.

The admin password protects the page even if someone finds the URL. The PAT is kept in session storage (cleared when you close the tab/app) and must be re-entered each time you come back — by design, so the token isn't sitting around in local storage.

---

## Step 4: Add the admin page to your home screen (optional)

You can add `admin.html` to your home screen the same way as the main app. Or just bookmark it. Or leave it and only visit it when needed — it's linked from the bottom of the main app ("+ Add Recipe" button).

---

## The new workflow for adding recipes

1. In Claude chat: "Can you give me a recipe for [X]?" — Claude generates it
2. Say "give me the JSON for my binder"
3. Claude gives you a JSON block — copy it
4. Open the admin page, paste, tap **Save to GitHub**
5. Wait ~1 minute for GitHub Pages to rebuild
6. Force-close and reopen the app — new recipe appears

---

## Recipe JSON format (for reference)

Each recipe is an object like this:

```json
{
  "id": "r4",
  "emoji": "🍝",
  "tag": "Dinner",
  "mealType": "Dinner",
  "title": "Cacio e Pepe",
  "description": "Roman pasta perfection in three ingredients.",
  "baseServings": 2,
  "ingredients": [
    { "id": "i1", "name": "spaghetti", "amount": 8, "unit": "oz" },
    { "id": "i2", "name": "pecorino romano, finely grated", "amount": 1, "unit": "cup" },
    { "id": "i3", "name": "black pepper, coarsely ground", "amount": 2, "unit": "tsp" }
  ],
  "steps": [
    { "id": "s1", "title": "Boil pasta", "content": "Cook until al dente.", "timer": 480 }
  ],
  "notes": "Reserve lots of pasta water — it's the sauce."
}
```

The admin page validates this automatically and will warn you if anything's missing.

---

## Security notes

- **The admin page is public** (anyone with the URL can see the password prompt), but:
  - Without the password, they can't get in
  - Without your PAT, they can't commit anything even if they bypass the password
  - The PAT is only scoped to your recipe-binder repo
- **The admin password** is hashed locally (SHA-256) and never transmitted. It only protects the UI.
- **The PAT** is stored in `sessionStorage` (not localStorage) so it clears when you close the tab
- Worst-case scenario: if someone somehow got your PAT, they could modify recipes in that one repo. They can't touch anything else in your GitHub account.

---

## Troubleshooting

**"Token rejected (401)"** — Your PAT doesn't have write access to the repo. Double-check it has "Contents: Read and write" and is scoped to the correct repository.

**"Could not fetch recipes.json"** — Either the repo/branch/path is wrong in the admin settings, or the token doesn't have access. Tap "Reset Settings" on the unlock screen and redo setup.

**New recipe doesn't appear in the app** — GitHub Pages takes ~1 minute to rebuild after a commit. Also make sure you force-close the PWA on your phone (swipe up in the app switcher) and reopen it. The service worker is set to always fetch `recipes.json` fresh when online, so this should just work.

**Admin page won't load offline** — Correct, by design. You need internet to commit to GitHub anyway.

---

## Updating the app code itself

If I ever need to update `index.html`, `admin.html`, or `sw.js` (different from adding recipes), you'll update those via the regular GitHub web UI — same as before. Bump `CACHE_NAME` in `sw.js` (e.g., `v2` → `v3`) to force the service worker to pick up changes.
