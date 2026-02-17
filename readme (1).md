# 🏦 Nova IT Operations Gantt Board

> A collaborative, real-time project tracking board for the Nova Microfinance IT Operations team — hosted on GitHub Pages and backed by Google Sheets.

![Status](https://img.shields.io/badge/status-live-brightgreen)
![Host](https://img.shields.io/badge/hosted-GitHub%20Pages-blue)
![Backend](https://img.shields.io/badge/backend-Google%20Apps%20Script-orange)
![Data](https://img.shields.io/badge/data-Google%20Sheets-34A853)

---

## 🔗 Live Board

**→ [https://chrl3y.github.io/IT_OPS_GANTT/](https://chrl3y.github.io/IT_OPS_GANTT/)**

> Access requires a name and the shared team key. Contact the IT Ops lead to be onboarded.

---

## 📋 Overview

The Nova IT Ops Gantt is a fully interactive project management board designed for the Nova Microfinance development sprint (Feb 17–23, 2026). It tracks issues across core banking, HUB reconciliation, Mifos X, and reporting systems.

All changes are synced live to a central Google Sheet, with full change history, per-task comments, and user attribution — no database or server required.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🗓️ **Interactive Gantt** | Drag bars to reschedule, resize to adjust duration |
| 🔄 **Live Sync** | Every edit pushes instantly to Google Sheets |
| 👥 **Collaboration** | Multiple users, all changes attributed by name |
| 💬 **Comments** | Per-task comment threads stored in Sheets |
| 📜 **Audit History** | Every field change logged with old/new values |
| 🔒 **Access Control** | Secret-key login — no Google OAuth required |
| 🌙 **Dark / Light Mode** | Theme toggle, preference saved locally |
| 📊 **Priority Filtering** | Filter by Critical / High / Medium / Enhancement / Design |
| 📤 **Export** | Download full board data as JSON |
| 📱 **Responsive** | Works on desktop and tablet |

---

## 🏗️ Architecture

```
┌─────────────────────────┐        fetch()        ┌──────────────────────────┐
│                         │ ─────────────────────▶ │                          │
│   GitHub Pages          │                        │   Google Apps Script     │
│   index.html            │ ◀───────────────────── │   Web App (REST API)     │
│                         │      JSON response     │                          │
└─────────────────────────┘                        └────────────┬─────────────┘
                                                                │
                                                                │ Sheets API
                                                                ▼
                                              ┌─────────────────────────────────┐
                                              │        Google Sheets            │
                                              │  ┌──────────────────────────┐  │
                                              │  │ Tab 1: Tasks             │  │
                                              │  │ Tab 2: History           │  │
                                              │  │ Tab 3: Comments          │  │
                                              │  └──────────────────────────┘  │
                                              └─────────────────────────────────┘
```

### Stack
- **Frontend:** Vanilla HTML / CSS / JavaScript — zero frameworks, zero dependencies
- **Backend:** Google Apps Script deployed as a Web App
- **Database:** Google Sheets (3 tabs)
- **Hosting:** GitHub Pages (free, CDN-backed)
- **Auth:** Shared secret key validated server-side

---

## 📁 Repository Structure

```
IT_OPS_GANTT/
│
├── index.html          # Full Gantt application (single file)
└── README.md           # This file
```

> The Apps Script backend (`Code.gs`) lives in the Google Sheet, not in this repo.

---

## 🗄️ Google Sheets Schema

### Tab 1 — `Tasks`
| Column | Type | Description |
|---|---|---|
| `id` | String | Unique issue ID e.g. `ISS-001` |
| `type` | String | `domain` for section headers, blank for tasks |
| `domain` | String | Domain group e.g. `CORE BANKING` |
| `name` | String | Issue title |
| `priority` | String | `critical` / `high` / `medium` / `enhancement` / `design` |
| `status` | String | e.g. `Escalated`, `Open`, `New` |
| `start` | Number | Start day index (0 = Monday, 0.5 = Monday afternoon) |
| `dur` | Number | Duration in days (0.25 = 2 hours) |
| `system` | String | Affected system/component |
| `fix` | String | Required fix description |
| `impact` | String | Business impact description |
| `owners` | JSON | Array of owner names e.g. `["Leo","Zayyad"]` |
| `updatedAt` | ISO Date | Last update timestamp (server-set) |
| `updatedBy` | String | Name of last editor (server-set) |

### Tab 2 — `History`
Append-only audit log. Every field change, creation, deletion, and comment is recorded with `timestamp`, `taskId`, `action`, `changedBy`, `fieldName`, `oldValue`, `newValue`, `snapshot`.

### Tab 3 — `Comments`
Per-task comment threads with `id`, `taskId`, `author`, `text`, `timestamp`, `edited`.

---

## 🔌 API Reference

The Apps Script Web App exposes a simple REST-style API.

### GET Endpoints
```
?action=ping&key=KEY                          → Health check
?action=getTasks&key=KEY                      → All task data
?action=getHistory&key=KEY&taskId=ISS-001     → History for a task
?action=getComments&key=KEY&taskId=ISS-001    → Comments for a task
```

### POST Endpoints
```json
{ "action": "saveTask",     "task": {...},          "key": "KEY", "user": "Leo" }
{ "action": "saveTasks",    "tasks": [...],         "key": "KEY", "user": "Leo" }
{ "action": "deleteTask",   "taskId": "ISS-001",    "key": "KEY", "user": "Leo" }
{ "action": "addComment",   "taskId": "ISS-001",    "text": "...", "key": "KEY", "user": "Leo" }
{ "action": "deleteComment","commentId": "CMT-abc", "key": "KEY", "user": "Leo" }
```

---

## 🚀 Deployment

### Prerequisites
- Google account with access to the Nova Microfinance Google Sheet
- GitHub account with write access to this repository

### Apps Script Backend
1. Open the Google Sheet → **Extensions → Apps Script**
2. Paste `Code.gs` contents
3. Set `SHEET_ID` and `SECRET_KEY` at the top of the file
4. Run `setupSheets()` once to create the 3 tabs
5. **Deploy → New deployment → Web app**
   - Execute as: `Me`
   - Who has access: `Anyone`
6. Copy the `/exec` URL

### Frontend
1. Paste the `/exec` URL and `SECRET_KEY` into the `CONFIG` block in `index.html`
2. Commit and push to the `main` branch
3. Enable GitHub Pages: **Settings → Pages → main → / (root)**

### Re-deploying After Code Changes
> ⚠️ After editing `Code.gs`, always create a **New Deployment** — not "Manage deployments → Edit". The exec URL will change; update `index.html` accordingly.

---

## 👥 Team Access

To give a teammate access to the board:
1. Share the Google Sheet with their Google account (Editor access)
2. Send them the board URL and `SECRET_KEY` via a **secure channel** (Signal, WhatsApp — not email)
3. They log in with their own name + the shared key

---

## 🔒 Security Notes

- The `SECRET_KEY` is embedded in `index.html` — **keep this repository private** or rotate the key regularly
- For production, inject the key via a GitHub Actions secret at build time instead of hardcoding it
- The Apps Script runs as the Sheet owner's account — treat the key with the same care as a password
- All write operations are logged in the `History` tab with the user's name and timestamp

---

## 🛠️ Local Development

No build step needed — it's a single HTML file.

```bash
# Clone the repo
git clone https://github.com/Chrl3y/IT_OPS_GANTT.git
cd IT_OPS_GANTT

# Open directly in browser
open index.html
# or
python3 -m http.server 8080   # then visit http://localhost:8080
```

> Note: `fetch()` calls to Apps Script will work from localhost since Apps Script doesn't enforce CORS for GET requests. POST requests may need the live GitHub Pages URL.

---

## 📌 Issue Priority Legend

| Badge | Priority | Meaning |
|---|---|---|
| 🔴 CRITICAL | `critical` | Blocks operations — fix within hours |
| 🟡 HIGH | `high` | Significant impact — fix within 1–2 days |
| 🔵 MEDIUM | `medium` | Important but non-blocking |
| 🟣 ENHANCEMENT | `enhancement` | Improvement to existing functionality |
| 🩵 DESIGN | `design` | Design / Phase 2 — requires spec before dev |

---

## 📄 License

Internal tool — Nova Microfinance IT Operations. Not for public distribution.

---

*Built and maintained by the Nova IT Operations team · Feb 2026*
