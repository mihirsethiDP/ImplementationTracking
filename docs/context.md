# Digital Paani — Feature Adoption Console
## Project Context File

---

## What This Project Is

A dashboard that shows feature adoption across all Digital Paani plants. It reads live from a Google Sheet via Apps Script and lets admins edit feature flags directly from the website. Deployed on GitHub Pages: **broadcast** at https://mihirsethidp.github.io/ImplementationTracking/ and **admin** at `/admin.html`.

> **Note on this doc:** the *technical* sections (Files, Column mapping, Apps Script, Plant status) are current as of 2026-09. The later UI-description sections (Website Features, Gap analysis, Summary stats, Theme) describe the **original** design and have since changed — the live tool now shows an **overall implementation %** (implemented ÷ total features) at plant & workspace level (the old client-vs-operator "gap" was removed), a dedicated table page, and type-ahead search. For current UI/behaviour, trust the code and [README.md](../README.md).

---

## Files

| File | Purpose |
|---|---|
| `index.html` | **Broadcast** site — view-only, no admin/edit path (public link, share widely) |
| `admin.html` | **Admin** site — same UI plus Edit Mode (password-gated); keep this URL private |
| Google Apps Script (in Google, not the repo) | Backend API for reading/writing the sheet |

> The two HTML files share the same UI — when changing shared behaviour, edit BOTH.
> `admin.html` = `index.html` + auth modal + Edit Mode toggle + the write path.

---

## Google Sheet

**URL:** https://docs.google.com/spreadsheets/d/1TS1EfvtoI7d3M2uyDCEx9LbkU1jkghzRaNdXowdJDiM/edit?usp=sharing

**Sheet name:** `Sheet1`

**Column mapping (current — updated 2026-09):**

> ⚠️ An **Active/Inactive** column was inserted at **column C** in 2026-09, which
> **shifted every following column +1**. The Apps Script `COL` map must match this
> exact order. **Add any future columns at the far right (after the last one) — never
> insert mid-sheet**, or the mapping silently breaks until `COL` is updated.

| Column | Field |
|---|---|
| A (1) | Workspace |
| B (2) | Plant |
| C (3) | **Active/Inactive** (plant status — text: `Active` / `Inactive`; blank = active) |
| D (4) | Visualiation |
| E (5) | Insights |
| F (6) | Dashboard |
| G (7) | Inventory |
| H (8) | Tickets |
| I (9) | Maintenance (discontinued — never read/written) |
| J (10) | Task List |
| K (11) | Data Input |
| L (12) | Remote Control |
| M (13) | Floc Detector |
| N (14) | Events |
| O (15) | OCR Data Input |
| P (16) | Reports (Daily/Weekly/Monthly/All/None) |
| Q (17) | Insight Digest(Whatsapp) |
| R (18) | Dashboard Summary(Whatsapp) |

**ChangeLog tab** — automatically created by Apps Script. Logs every edit with timestamp, row, field, value, and editor email.

---

## Apps Script

**Deployed URL (current — updated 2026-09, public form):**
```
https://script.google.com/macros/s/AKfycbxFSHNmPjCEtZ8NLNNhGjQKQWJUCjzObmGXwiza8TPR88vfHGwUwstV6gU0lBaAV8elfA/exec
```
Set in `const API` in both `index.html` and `admin.html`. (Older `/a/macros/digitalpaani.com/s/…/exec` form is retired.)

**Deployment settings (must stay this way, or the tool can't load):**
- Execute as: **Me**
- Who has access: **Anyone** — required for the no-login public broadcast. "Anyone with a Google account" forces a login → browsers can't fetch it (CORS on the login redirect) → "Failed to fetch".
- Setting access to *Anyone* changes the URL to the `/macros/s/…/exec` form (above). If the URL ever changes again, update `const API` in both HTML files.
- Reading `Sheet1` by name: if the tab is renamed, `getSheetByName` returns null → "Cannot read properties of null (reading 'getLastRow')". Keep the tab named **Sheet1**.

**Key technical decision — why `getDisplayValues()` instead of `getValues()`:**
The Workspace column uses merged cells in Google Sheets. `getValues()` only returns the value in the top cell of a merge, leaving all other rows empty. `getDisplayValues()` returns what is visually shown in every cell, so merged cells return the same workspace value for every row they span — no forward-fill needed.

**API endpoints:**

| Method | Params | Purpose |
|---|---|---|
| GET | (none) | Returns all plant records as JSON: `{success, records:[{rowIndex, workspace, plant, reportsType, Status, <feature booleans…>}]}` |
| GET | `?action=validate&pw=XXX` | NOT USED — admin password is checked client-side in `admin.html` |
| POST | `{rowIndex, field, value}` | Updates a single boolean feature or the Reports cadence (admin only). Status is read-only via the API. |

The dashboard **discovers features dynamically** from the record keys (everything except
`rowIndex, workspace, plant, reportsType`, and the auto-detected status column). The
frontend loads with **3× auto-retry** to ride out transient Apps Script hiccups.

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

## Plant status — Active / Inactive (added 2026-09)

- Column **C** holds each plant's status as text: **`Active`** or **`Inactive`** (blank = treated as active). `Inactive` = churned / contract ended / no longer serviced.
- The Apps Script returns it under the key **`Status`**; the dashboard **auto-detects** the status column by its values (so the header name doesn't have to be exactly "Status"), and treats it as **status, not a feature** (excluded from feature counts).
- **Inactive plants are excluded by default** from every metric, the table, and workspace roll-ups. A **Status filter** (Active only / Inactive only / All) is in the filter bar; inactive plants are badged.

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

## Data — ~101 Plants (92 active + 9 inactive), ~67 Workspaces (grows over time)

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

