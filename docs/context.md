# Digital Paani — Feature Adoption Console
## Project Context File

---

## What This Project Is

A single-file HTML dashboard that shows feature adoption across all Digital Paani plants. It reads live from a Google Sheet via Apps Script and lets admins edit feature flags directly from the website.

---

## Files

| File | Purpose |
|---|---|
| `digital-paani-feature-dashboard.html` | The entire website — single self-contained file |
| `digital_paani_apps_script.gs` | Google Apps Script — backend API for reading/writing Google Sheet |

---

## Google Sheet

**URL:** https://docs.google.com/spreadsheets/d/1TS1EfvtoI7d3M2uyDCEx9LbkU1jkghzRaNdXowdJDiM/edit?usp=sharing

**Sheet name:** `Sheet1`

**Column mapping:**

| Column | Field |
|---|---|
| A (1) | Workspace |
| B (2) | Plant |
| C (3) | Visualiation |
| D (4) | Insights |
| E (5) | Dashboard |
| F (6) | Inventory |
| G (7) | Tickets |
| H (8) | Maintenance (discontinued — never written back) |
| I (9) | Task List |
| J (10) | Data Input |
| K (11) | Remote Control |
| L (12) | Floc Detector |
| M (13) | Events |
| N (14) | OCR Data Input |
| O (15) | Reports (Daily/Weekly/Monthly/All/None) |
| P (16) | Insight Digest(Whatsapp) |
| Q (17) | Dashboard Summary(Whatsapp) |

**ChangeLog tab** — automatically created by Apps Script. Logs every edit with timestamp, row, field, value, and editor email.

---

## Apps Script

**Deployed URL:**
```
https://script.google.com/a/macros/digitalpaani.com/s/AKfycbxFSHNmPjCEtZ8NLNNhGjQKQWJUCjzObmGXwiza8TPR88vfHGwUwstV6gU0lBaAV8elfA/exec
```

**Deployment settings:**
- Execute as: Me
- Who has access: Anyone (currently open — can restrict to digitalpaani.com accounts)

**Key technical decision — why `getDisplayValues()` instead of `getValues()`:**
The Workspace column uses merged cells in Google Sheets. `getValues()` only returns the value in the top cell of a merge, leaving all other rows empty. `getDisplayValues()` returns what is visually shown in every cell, so merged cells return the same workspace value for every row they span — no forward-fill needed.

**API endpoints:**

| Method | Params | Purpose |
|---|---|---|
| GET | (none) | Returns all plant records as JSON |
| GET | `?action=validate&pw=XXX` | NOT USED — password now hardcoded in HTML |
| POST | `{rowIndex, field, value}` | Updates a single field in the sheet |

---

## Feature Categories

**Client-side features (3):**
- Reports — category field: Daily / Weekly / Monthly / All / None (default: None)
- Insight Digest(Whatsapp) — boolean
- Dashboard Summary(Whatsapp) — boolean

**Operator-side features (3):**
- OCR Data Input — boolean
- Data Input — boolean
- Task List — boolean

**Common features (8):**
- Visualiation, Insights, Dashboard, Inventory, Tickets, Remote Control, Floc Detector, Events — all boolean

**Discontinued:**
- Maintenance — excluded from all analytics and never written back to sheet

---

## Auth / Access

**Viewer mode:** Anyone who opens the HTML file can view all analytics. No login required.

**Admin (Edit) mode:**
- Click ✏ Edit Mode button above the plant-level detail table
- Modal popup appears with password field (👁 show/hide button included)
- Password: `digitalpaani@123` — hardcoded in HTML (safe because sheet access is already restricted)
- Correct password → badge flips from VIEWER to ADMIN, dots become clickable
- Wrong password → field shakes, shows error message
- Closing tab / refreshing → returns to VIEWER mode (session only)

**Why password is in HTML (not server-side):**
Previous attempts to validate via Apps Script POST failed due to Google's redirect stripping the request body. GET-based validation also had issues. Since the Google Sheet itself is already restricted to @digitalpaani.com accounts, hardcoding in HTML is safe for this use case.

---

## Website Features

### Global filters (top of page)
- Workspace dropdown
- Plant name search
- Feature dropdown
- Report frequency dropdown (Daily/Weekly/Monthly/All/None)
- Reset button

### Summary stats strip
- Plants in view, Workspaces, Client adoption %, Operator adoption %, Zero-feature plants

### Gap analysis
- Client-side: Reports frequency stacked bar + boolean feature bars
- Operator-side: feature bars
- Summary card showing which side is ahead and by how many points

### Feature adoption grid
- One card per feature showing adoption % and count
- Reports card shows frequency distribution as stacked coloured bar
- Click any card to filter the table below

### Plant-level detail table
- Two search bars: plant name + workspace name (table-local, independent of global filters)
- Edit Mode toggle + VIEWER/ADMIN badge (same row, right side)
- Reports shown as frequency pill (Daily/Weekly/Monthly/All/None)
- Boolean features shown as green/grey dots
- Coverage score per plant (e.g. 7/14)
- Export CSV button

### Drill-down navigation
**Workspace page** (click workspace name in table):
- Navy header with workspace stats (total plants, active, client %, operator %)
- Grid of plant cards showing coverage bar, feature dots, report frequency badge
- Click any plant card to go to plant page

**Plant page** (click plant name in table or from workspace page):
- Navy header with plant name, workspace label, key stats
- SVG coverage ring showing overall % 
- Three feature group cards: Client-side, Operator-side, Common — each showing every feature with ON/OFF badge

**Navigation:**
- Breadcrumb trail at top: Home › Workspace › Plant
- Back button (goes to workspace if came from there, home otherwise)
- Home button

---

## Theme & Design

- Primary colour: `#193458` (navy)
- Font: Inter (UI) + JetBrains Mono (numbers, labels, codes)
- Logo: Digital Paani logo embedded as base64 PNG in the HTML header
- Colour coding:
  - Teal `#0c8a7a` — client-side features, ON state
  - Amber `#c97a2c` — operator-side features
  - Navy `#193458` — common features, UI chrome
  - Red `#b3473f` — errors, low coverage
  - Green `#1a7a4a` — save confirmation

---

## Data — 96 Plants, 41+ Workspaces

Notable workspaces include:
- GAJWEL PRAGNAPUR MUNICIPALITY (3 plants)
- Unassigned (plants with no workspace in sheet — should be 0 after workspace fix)

**Workspace reading fix history:**
- v1/v2: Used `getValues()` + forward-fill — failed for first 10 plants (no merged cell above them)
- v3+: Uses `getDisplayValues()` — correctly reads merged cells for all rows

---

## How to Update / Redeploy

**If you edit the Apps Script:**
Deploy → Manage deployments → Edit (pencil) → Version: New version → Deploy

**If you add new plants to the sheet:**
Just click ↻ Refresh on the website — data reloads live from the sheet

**If you want to change the admin password:**
Open the HTML file in a text editor → find `const ADMIN_PASSWORD = "digitalpaani@123"` → change the value → save

**If you want to restrict viewing to @digitalpaani.com only:**
Apps Script → Deploy → Manage deployments → Edit → Who has access: "Anyone with Google account" → New version → Deploy
Note: this will make the site show no data for users not logged into a digitalpaani.com account

---

## Known Limitations

- Two admins editing at the same time → last write wins (no conflict detection)
- Session only — closing the tab resets to VIEWER mode, password required again
- Maintenance column is permanently excluded (discontinued feature)
- Reports field only supports: Daily, Weekly, Monthly, All, None — any other value defaults to None

