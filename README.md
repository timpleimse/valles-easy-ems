# Valles EASY EMS — Complete Setup Guide

Single-file web app. No server, no database. It runs entirely in the browser
and syncs its data to a folder in a Google Drive account you control.

## Account split used in this setup

| Purpose | Account |
|---|---|
| GitHub repo + GitHub Pages hosting | **timple.imse@gmail.com** |
| Google Cloud project, OAuth client, Drive folder, Sheet | **vallesindia@gmail.com** |

This split is fine — the two never need to know about each other. GitHub only
serves the HTML file; the OAuth client decides who may sign in; and whichever
account signs in inside the app is where the data physically lives.

**The one rule to remember:** the account you pick in the "Connect Google Drive"
popup is the account that gets the data. Always pick **vallesindia@gmail.com**
there.

---

# PART A — Publish the app on GitHub Pages
*(signed in as **timple.imse@gmail.com**)*

### A1. Create the repository
1. Go to [github.com](https://github.com) → **+** (top-right) → **New repository**.
2. Repository name: `valles-easy-ems`
3. Visibility: **Public** ← required; GitHub Pages is disabled on private repos for free accounts.
4. Click **Create repository**.

### A2. Upload the app as `index.html`
1. On your local machine, **rename `valles-easy-ems.html` to `index.html`**.
   (GitHub Pages serves `index.html` automatically at the root URL.)
2. In the repo: **Add file → Upload files** → drag `index.html` in → **Commit changes**.

### A3. Turn on Pages
1. Open the **repository's own Settings tab** — the one in the repo's toolbar
   (Code · Issues · Pull requests · Actions · … · **Settings**).
   Direct link: `https://github.com/timpleimse/valles-easy-ems/settings/pages`
   > This is **not** `github.com/settings/pages` — that's your personal account
   > settings and only manages custom domains. Wrong page.
2. Left sidebar → **Pages** (under "Code and automation").
3. **Build and deployment** → Source: **Deploy from a branch**
4. Branch: **main**, folder: **/ (root)** → **Save**.
5. Wait ~1 minute, refresh. It shows your live URL:

   ```
   https://timpleimse.github.io/valles-easy-ems/
   ```

### A4. Write down your ORIGIN
From that URL, the **origin** is just the scheme + host:

```
https://timpleimse.github.io
```

No repo path. No trailing slash. You need this exact string in Part B.

---

# PART B — Google Cloud setup
*(signed in as **vallesindia@gmail.com** — check the avatar in the top-right of Cloud Console before every step)*

### B1. Create the project
1. Open [console.cloud.google.com](https://console.cloud.google.com/).
2. Confirm the top-right avatar shows **vallesindia@gmail.com**. If not, switch
   accounts (or open an Incognito window and sign in as that account — simplest
   way to avoid mixing up two logged-in Google accounts).
3. Project dropdown (top bar) → **New Project**.
4. Name: `Valles EASY EMS` → **Create**. Make sure it's selected afterwards.

### B2. Enable the two APIs
**APIs & Services → Library**, search and **Enable** each:
- **Google Drive API**
- **Google Sheets API**

### B3. Configure the OAuth consent screen
**APIs & Services → OAuth consent screen**
1. User type: **External** → **Create**.
2. App name: `Valles EASY EMS`
   User support email: `vallesindia@gmail.com`
   Developer contact email: `vallesindia@gmail.com` → **Save and Continue**.
3. **Scopes** — you can skip this page entirely (the app requests its scopes at
   runtime). → **Save and Continue**.
4. **Test users** → **Add users** → add:
   - `vallesindia@gmail.com`  ← **essential, this is the account that will sign in**
   - (optionally `timple.imse@gmail.com` if you want to be able to test it too)
   → **Save and Continue** → **Back to Dashboard**.

   > While the app is in **Testing** mode, *only* the listed test users can sign
   > in. Anyone else gets "access blocked". This is exactly what you want for an
   > internal company tool.

### B4. Create the OAuth Client ID
**APIs & Services → Credentials → + Create Credentials → OAuth client ID**
1. Application type: **Web application**
2. Name: `Valles EASY EMS Web`
3. **Authorized JavaScript origins** → **+ Add URI**:
   ```
   https://timpleimse.github.io
   ```
   > Origin only — no `/valles-easy-ems`, no trailing slash. This is the single
   > most common cause of `redirect_uri_mismatch` / `origin not allowed` errors.
4. **Authorized redirect URIs** → leave **empty**. This app uses the JavaScript
   token flow, which doesn't use redirect URIs.
5. **Create** → copy the **Client ID** (ends in `.apps.googleusercontent.com`).
   You can reopen it any time from the Credentials list.

*(Optional, for local testing: also add `http://localhost:8000` as a second
authorized origin.)*

---

# PART C — Connect the app
*(in your browser, at the live GitHub Pages URL)*

1. Open `https://timpleimse.github.io/valles-easy-ems/`
2. Go to the **Settings** tab → unlock with the admin password.
3. Scroll to **Google Drive Backup & Sync**.
4. Paste the Client ID → **Save Client ID**.
5. Click **Connect Google Drive**.
6. **In the account chooser popup, pick `vallesindia@gmail.com`.**
   ⚠️ Your browser is likely signed into `timple.imse@gmail.com` too — if you
   pick the wrong one, the data folder is created in the wrong Drive. If you're
   unsure, use "Use another account" and sign in explicitly.
7. Google shows **"Google hasn't verified this app"** — expected, since this is
   your own private OAuth client in Testing mode.
   → **Advanced** → **Go to Valles EASY EMS (unsafe)** → **Continue**.
8. Approve the Drive + Sheets permissions.

Done. In `vallesindia@gmail.com`'s Drive you'll now find:

```
📁 Valles EASY EMS Data
   ├── 📄 data.json          ← live copy, auto-synced after every change
   └── 📁 Backups
        └── backup-2026-07-17T....json   ← from "Backup Now to Drive"
```

- **Backup Now to Drive** — saves a timestamped snapshot; the app keeps only the
  **latest 5** and auto-deletes older ones.
- **Restore** — pick any of those 5 in Settings and confirm; it overwrites all
  current data in the app.
- **Export to Google Sheet** — creates/updates a spreadsheet in the same folder
  with readable **Employees** and **Payslips** tabs. Good for sharing with your
  accountant without giving them the app.

---

# Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| Cloud Console has no Source dropdown under Pages | You're on `github.com/settings/pages` (account). Use the **repo's** Settings → Pages. |
| Pages Source dropdown greyed out | Repo is Private. Make it Public. |
| 404 at the Pages URL | File isn't named `index.html`, or it's in a subfolder instead of root. |
| "Access blocked: app has not completed verification" | Signing-in account isn't in **Test users**. Add it in the consent screen. |
| "Error 400: origin_mismatch" / "not allowed" | Authorized JavaScript origin doesn't exactly match. Must be `https://timpleimse.github.io` — no path, no trailing slash. Changes can take a few minutes to propagate. |
| Sync silently stops after ~an hour | Normal — the Google token expired. Click **Connect Google Drive** again. Nothing is lost; the local browser copy re-syncs. |
| Data went to the wrong Drive account | You picked the wrong account in the popup. Disconnect, reconnect, choose `vallesindia@gmail.com`, then **Backup Now**. Delete the stray folder from the other Drive. |

### Notes
- Scope is `drive.file` — the app can only see and edit **files and folders it
  created itself**. It cannot read anything else in that Drive account.
- The Client ID is not a secret; it's safe to have it visible in a public repo
  or in the app UI. The authorized-origin restriction is what protects it.
- Updating the app later: edit/replace `index.html` in the repo and commit —
  Pages redeploys in a minute or two. Your Drive data is untouched.
- The **Backup** tab still offers manual JSON download/upload, independent of
  Google Drive. Worth doing once before any big change.
