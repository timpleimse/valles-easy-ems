# Valles EASY EMS — Deployment Guide

This is a single-file web app (`valles-easy-ems.html`). It needs no server or
database — it runs entirely in the browser and (optionally) syncs its data to
a folder in your own Google Drive.

---

## Part 1 — Deploy to GitHub Pages

1. Create a new **public** GitHub repository (e.g. `valles-easy-ems`).
2. Upload `valles-easy-ems.html` to the repo, and **rename it to `index.html`**
   (GitHub Pages serves `index.html` by default at the root URL).
3. In the repo, go to **Settings → Pages**.
4. Under "Build and deployment", set **Source: Deploy from a branch**,
   **Branch: main**, folder **/ (root)** → **Save**.
5. Wait ~1 minute, then your app is live at:
   `https://<your-username>.github.io/<repo-name>/`
6. Keep this exact URL handy — you'll need it in Part 2.

> Any time you edit `index.html`, commit + push and GitHub Pages updates
> automatically within a minute or two.

---

## Part 2 — One-time Google Cloud setup (for Drive sync)

The app talks to Google Drive using **your own** OAuth Client ID, so only you
control access — Anthropic/Claude has nothing to do with this at runtime, and
neither does anyone else.

1. Go to [console.cloud.google.com](https://console.cloud.google.com/) and
   create a new project (or reuse one).
2. **APIs & Services → Library** — enable:
   - **Google Drive API**
   - **Google Sheets API**
3. **APIs & Services → OAuth consent screen**:
   - User type: **External** (unless you have a Workspace org, then Internal).
   - Fill in app name, your email, etc. Add your own Google account under
     **Test users** (while the app is unpublished/in testing, only test users
     can sign in — that's fine for internal company use).
4. **APIs & Services → Credentials → Create Credentials → OAuth client ID**:
   - Application type: **Web application**
   - Name: anything, e.g. "Valles EASY EMS"
   - **Authorized JavaScript origins**: add your GitHub Pages origin, e.g.
     `https://<your-username>.github.io`
     (origin only — no path, no trailing slash)
   - Leave "Authorized redirect URIs" empty (not needed for this flow).
   - Click **Create** — copy the **Client ID** (ends in
     `.apps.googleusercontent.com`).

---

## Part 3 — Connect the app to Drive

1. Open your deployed app → **Settings** tab (unlock with your admin password).
2. Scroll to **Google Drive Backup & Sync**, paste the Client ID, click
   **Save Client ID**, then **Connect Google Drive**.
3. Sign in with the Google account you added as a test user, and approve
   access. (You'll see a Google "unverified app" warning since it's your own
   personal OAuth client in testing mode — click Advanced → Go to
   [app name] to proceed. This is expected and safe; you're the developer.)

Once connected, the app automatically creates a **"Valles EASY EMS Data"**
folder in your Drive, with:
- `data.json` — the live, always-current copy of all your data (auto-synced
  a couple of seconds after every change you make in the app).
- A **Backups** subfolder — click **"Backup Now to Drive"** any time to save
  a timestamped snapshot. The app automatically keeps only the **latest 5**
  backups and deletes older ones.
- **"Export to Google Sheet"** creates/updates a spreadsheet with readable
  Employees and Payslips tabs, handy for sharing with your accountant.

### Restoring a backup
In Settings, under "Last 5 Backups in Drive", click **Restore** next to any
backup and confirm. This overwrites all current data in the app with that
backup's contents.

### Notes
- The app only has permission to see files/folders **it creates itself**
  (Drive scope `drive.file`) — it cannot browse or touch the rest of your
  Drive.
- The Google sign-in session lasts about an hour; if a sync silently stops
  working, just click **Connect Google Drive** again to refresh it — no data
  is lost, your local browser copy stays authoritative until it re-syncs.
- You can still use the separate **Backup** tab any time for a manual
  download/upload of a `.json` file — that works with or without Google
  Drive connected.
