# Roadmap 2026-2027 - Hosting Guide (Live Sync Edition)

## What is in this folder
- `index.html` - the calendar app: password gate, live sync, and in-app editing built in
- `manifest.json` + `icon.png` - iPhone home-screen app support
- `sw.js` - service worker (fast launch + offline)
- `logo.png` - gate page logo

## Step 1 - Firebase (makes it live for everyone)
Without this step the app still works, but check-offs and edits stay on each device (the header shows "standalone"). With it, everything syncs live to all viewers.

1. Go to console.firebase.google.com -> Add project (any name, Analytics off is fine).
2. In the project: click the web icon (`</>`) -> register an app -> copy the `firebaseConfig` object it shows.
3. Build -> Authentication -> Get started -> Sign-in method -> enable **Anonymous**.
4. Build -> Firestore Database -> Create database -> production mode -> pick a US region.
5. Firestore -> Rules tab -> replace with the rules below -> Publish:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /boards/{board}/{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

6. Edit `index.html`: find `const FIREBASE_CONFIG = null;` near the top of the main script and replace `null` with the config object from step 2.
7. Done. On first load the app seeds the database with the built-in calendar; from then on the database is the source of truth. The header shows a green dot and "live - synced".

## Step 2 - Publish on GitHub Pages
1. Create a repository, upload every file in this folder to the root.
2. Settings -> Pages -> Deploy from branch -> `main` / root -> Save.
3. Live at `https://<username>.github.io/<repo>/` after a minute.

## Using it
- **Password**: `RedSox$trength` (shared; remembered per device). To change it, see the comment above `PASS_HASH` in index.html.
- **Check-offs**: any checkbox (sidebar, cards, quick look) syncs to everyone within a second or two.
- **Editing**: tap **Edit** in the header. Then:
  - **+ Add Item** appears in the filter bar (single date or range, title, category, detail bullets)
  - every card shows **Date** (change the date/range) and **Remove** buttons
  - tap **Done Editing** to exit. Changes go live for everyone immediately - no re-upload ever needed.
- **iPhone app**: open in Safari -> Share -> Add to Home Screen. Full screen, stays past the gate, works offline.

## Updating the app itself (not the data)
Data never requires redeploying. Only if you change the app's code/design: replace index.html and bump `CACHE_VERSION` in sw.js (v2 -> v3).

## Security honesty
Anonymous auth + shared password is a courtesy lock, not real security. Anyone with the URL and mild determination could read or write the board. Keep medical, contractual, and personal player data out of it.
